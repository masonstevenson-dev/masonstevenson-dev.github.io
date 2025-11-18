---
permalink: /posts/2023-09-10-InputRoutingDeepDive/
layout: single
title:  "A (Semi) Comprehensive Breakdown of Unreal Engine Input Routing"
date:   2023-09-10 19:46:14 -0800
classes: wide
sidebar:
  nav: "docs"
categories: 
---

One major frustration I've had is understanding how UE's input routing system works. Earlier this year, I took a crack at trying to understand how it works end-to-end. These are my findings.

When I say *input routing*, I'm really talking about two (related) topics:

1. How UE propagates user keypress events throughout its various systems.
2. How developers can map various keypress events to code.

Let's tackle #2 first, because it will help with understanding #1. At the time of writing, Unreal has 5 different ways to handle input mapping:

1. The debug way:
   * You can literally add a keypress node to a blueprint. So for example you could bind 'E' to some execution path.
   * Typically, you would only use this for a quick and dirty debug solution.


2. The [old way](https://docs.unrealengine.com/4.26/en-US/InteractiveExperiences/Input/):
   * You can go into the game's project settings (Project Settings -> Engine -> Input) and add a list of key bindings. These bindings create new action events you can branch from in Blueprint.


3. [Enhanced Input System](https://docs.unrealengine.com/5.2/en-US/enhanced-input-in-unreal-engine/):
   * This system pushes input bindings into data assets. There are two pieces:
     * The **input mapping context**, which is essentially a replacement for the key binding map. They map from key->input_action
     * The **input actions**, which define the actual events that you branch from.


4. [Common UI Input Action System](https://docs.unrealengine.com/5.3/en-US/common-ui-plugin-for-advanced-user-interfaces-in-unreal-engine/) (Note: The info for this section might be out of date- I researched this in Jan 2023):
   * The common UI input action system is a half-baked collection of systems that let you do a couple different things:
     * Bind user input to widget functionality
     * Display icons for input keys
     * That's basically it
   * CommonUI IAS consists of the the following:
     * **Input Action Table**
       * This contains bindings from [action name]->[device specific keybinding]. This is a *one-to-many* relationship. So you could bind the "jump" action to [spacebar] on a keyboard and [A] on an xbox controller.
     * **InputControllerData**
       * This basically just allows you to give each key on an input device (keyboard, controller, etc) an icon.
     * **CommonUIInputData**
       * Here you specify a universal "click" and "back" action. The action must link to a row in your input action table.


5. [FNavigationConfig](https://docs.unrealengine.com/5.3/en-US/API/Runtime/Slate/Framework/Application/FNavigationConfig/) black magic:
   * You can actually rebind the engine definitions for basic actions like a "click". See explanation below.

Okay, with this background info out of the way, let's dive into #1

## A (Semi) Comprehensive Breakdown of Unreal Engine Input Routing

<blockquote style="border-left: 4px solid darkgrey; background: #dfe2e5;"> <b>NOTE:</b> The following assumes you know at least a little bit about [Slate](https://docs.unrealengine.com/5.3/en-US/slate-overview-for-unreal-engine/). If you don't, the short explanation is that Slate is the O.G. UI framework that Unreal is built on. Slate has been superseded by [UMG](https://docs.unrealengine.com/5.3/en-US/umg-ui-designer-for-unreal-engine/); however, most UMG widgets are *actually* just wrapper classes for underlying slate widgets. An easy way to tell if you are looking at slate code (besides the weird-ass syntax) is that slate widgets all start with 'S': SWidget, SViewport, SButton, etc. If there is a UMG wrapper class for a slate widget, it will be named with a 'U' (UButton for example).</blockquote>

When the user presses a key, it will first be detected by some platform-specific code. So for example, if you press a keyboard key or a mouse button on Windows, your keypress will end up in the FWindowsApplication::ProcessDeferredMessage() fn inside WindowsApplication.cpp:
```cpp
int32 FWindowsApplication::ProcessDeferredMessage(const FDeferredWindowsMessage& DeferredMessage)
{
    // ...

    switch(msg)
    {
        // ...

        case WM_KEYDOWN:
        {
            // This is where your keypress is initially routed.
			const bool Result = MessageHandler->OnKeyDown(
            	ActualKey, CharCode, bIsRepeat );
        }
        // ...
        case WM_MOUSEWHEEL:
        {
            // This is where your mousewheel is initially routed.
            const BOOL Result = MessageHandler->OnMouseWheel(
                static_cast<float>( WheelDelta ) * SpinFactor, CursorPos);
        }
        // ...
    }
}
```

From there, the platform-specific code will trigger the appropriate fn on [FSlateApplication](https://docs.unrealengine.com/5.3/en-US/API/Runtime/Slate/Framework/Application/FSlateApplication/) via the [FGenericApplicationMessageHandler](https://docs.unrealengine.com/5.3/en-US/API/Runtime/ApplicationCore/GenericPlatform/FGenericApplicationMessageHandle-/) interface:
- OnKeyDown
- OnMouseDown
- OnMouseWheel
- etc.

From here, the actual routing begins. FSlateApplication will loop through all the widgets in the current widget hierarchy and give each one a chance to "handle" the input event. If the FReply is marked as "handled", the loop terminates.

```cpp
// inside FSlateApplication.cpp
bool FSlateApplication::ProcessKeyDownEvent( const FKeyEvent& InKeyEvent )
{
    // ...

    FReply Reply = FReply::Unhandled();

    // ...

    // This is where input is passed down the widget hierarchy.
    Reply = FEventRouter::RouteAlongFocusPath(...){
        // ...
    }
}
```

Now at this point, you may be thinking- *why are we even talking about widgets and UI elements? Isn't the input supposed to be captured by one of the input systems that was mentioned above?* Well as it turns out, no- the base system for handling your input is actually Slate, and LATER slate will hand off your input to the other various input handling systems. If you are running your game from the editor, your input will actually first be routed through a bunch of UE editor widgets, but eventually it will hit the game viewport (widget of type SViewport): ![](/assets/images/route_to_viewport.png)

From here, the viewport will call the appropriate fn (OnKeyDown, OnMouseWheel, etc) on the ViewportInterface, which by default should be of type FSceneViewport.

```cpp
FReply SViewport::OnKeyDown( const FGeometry& MyGeometry, const FKeyEvent& KeyEvent )
{
    // Call ViewportInterface OnKeyDown(),
    // which is actually FSceneViewport::OnKeyDown().
    return ViewportInterface.IsValid() ?
        ViewportInterface.Pin()->OnKeyDown(MyGeometry, KeyEvent) : FReply::Unhandled();
}
```

The SceneViewport does some input validation, and then passes the event to the ViewportClient:

```cpp
FReply FSceneViewport::OnKeyDown( const FGeometry& InGeometry, const FKeyEvent& InKeyEvent )
{
   // ...

    if (!ViewportClient->InputKey(
        FInputKeyEventArgs(
            this,
            InKeyEvent.GetInputDeviceId(),
            Key,
            InKeyEvent.IsRepeat() ? IE_Repeat : IE_Pressed,
            1.0f,
            false
        )
    ))
    {
        // ...
    }
    // ...
}
```

And here, finally, we get to the first of the input handling systems mentioned above. If you are using the CommonUI plugin, the [installation instructions](https://docs.unrealengine.com/5.3/en-US/common-ui-quickstart-guide-for-unreal-engine/) tell you to set the Game Viewport Client Class setting in the Unreal editor to CommonGameViewportClient. This is where CommonUI injects its input handling code. Similarly, if you want to define your own custom class that handles input at the earliest possible stage, you can create your own ViewportClient class that extends either UGameViewportClient or UCommonGameViewportClient.

Regardless of if you are using CommonUI or not, the input should then go to the GameViewportClient. The GameViewportClient is responsible for routing input to the PlayerController:

```cpp
// inside UGameViewportClient::InputKey()
if (!bResult)
{
    ULocalPlayer* const TargetPlayer = GEngine->GetLocalPlayerFromInputDevice(this, EventArgs.InputDevice);
    if (TargetPlayer && TargetPlayer->PlayerController)
    {
        bResult = TargetPlayer->PlayerController->InputKey(
            FInputKeyParams(
                EventArgs.Key,
                EventArgs.Event,
                static_cast<double>(EventArgs.AmountDepressed),
                EventArgs.IsGamepad(), EventArgs.InputDevice
            )
        );
    }

    // A gameviewport is always considered to have responded to a mouse buttons
    // to avoid throttling
    if (!bResult && EventArgs.Key.IsMouseButton())
    {
        bResult = true;
    }
}
```

Note that last bit about the mouse button always being marked as handled. This means that the GameViewport *always* captures mouse input, and therefore mouse input will not be routed to any further slate widgets in the scene. Normally, this doesn't seem to be an issue because for mouse events, the FSlateApplication will construct the input event path based on whatever widgets are currently underneath the mouse cursor (see calls to *LocateWindowUnderMouse* inside SlateApplication.cpp). However, I specifically ran into an issue with this where I was trying to use mousewheel events for up/down navigation, and I was confused to find that the GameViewport was silently eating my input. I ended up scrapping this plan and instead routed the mousewheel events through the PlayerController (via the EnhancedInput system) and then created functions for faking the navigation behavior that happens when you press a "real" navigation button like one of the arrow keys (I eventually abandoned UE's navigation system entirely, but this is a story for another day). 

Once we get to the PlayerController, this is where we are in more familiar territory. The input gets routed to the UPlayerInput object, which will be of type UEnhancedPlayerInput if the enhanced input system is enabled (search for UInputSettings::GetDefaultPlayerInputClass() in PlayerController.cpp if you want to see where this is initialized).

The input (if non-analog) can also be routed to the [XR](https://www.unrealengine.com/en-US/xr) system, which I know next to nothing about.

Now remember, at this point we are still technically just looping through all the widgets on the screen and passing the input event to them. We just took a very long detour on the SViewport widget, but none of the dependent subsystems (GameViewportClient, PlayerController, etc) actually mark the event as *handled*, UE will go on its merry way passing the input event around to the rest of the widgets in the widget hierarchy. This is where some of the more mysterious navigation stuff takes place.

### Navigation Config Black Magic

<div>The FSlateApplication maintains an FNavigationConfig which actually contains **hardcoded** values for navigation directions and the "accept" and "back" keys. This is how UE just magically "knows" about these inputs. The base SWidget class uses this to retrieve the binding from key->navigation direction. For example:</div>

```cpp
FReply SWidget::OnKeyDown(const FGeometry& MyGeometry, const FKeyEvent& InKeyEvent)
{
    if (bCanSupportFocus && SupportsKeyboardFocus())
    {
        EUINavigation Direction = FSlateApplicationBase::Get().GetNavigationDirectionFromKey(InKeyEvent);
        if (Direction != EUINavigation::Invalid)
        {
            const ENavigationGenesis Genesis = InKeyEvent.GetKey().IsGamepadKey() ? 
                ENavigationGenesis::Controller : ENavigationGenesis::Keyboard;

            // The navigation direction is attached to the input event FReply.
            return FReply::Handled().SetNavigation(Direction, Genesis);
        }
    }
    return FReply::Unhandled();
}
```

Here are the default bindings (found in NavigationConfig.cpp):

| Nav Enum                    | Key Binding                                          |
| --------------------------- | ---------------------------------------------------- |
| EUINavigation::Left         | EKeys::Left, EKeys::Gamepad_DPad_Left                |
| EUINavigation::Right        | EKeys::Right, EKeys::Gamepad_DPad_Right              |
| EUINavigation::Up           | EKeys::Up, EKeys::Gamepad_DPad_Up                    |
| EUINavigation::Down         | EKeys::Down, EKeys::Gamepad_DPad_Down                |
| EUINavigationAction::Accept | EKeys::Enter, EKeys::SpaceBar, EKeys::Virtual_Accept |
| EUINavigationAction::Back   | EKeys::Escape, EKeys::Virtual_Back                   |

As it turns out, you can actually override this behavior too. You can create your own custom NavigationConfig and then rebind or disable whatever keys you want. For example:

```cpp
EUINavigation FAnankeNavigationConfig::GetNavigationDirectionFromAnalog(
    const FAnalogInputEvent& InAnalogEvent)
{
    // This will disable using a gamepad's thumbstick for navigation.
    // You can still use the D-pad.
    return EUINavigation::Invalid;
}

EUINavigationAction FAnankeNavigationConfig::GetNavigationActionForKey(const FKey& InKey) const
{
    if (InKey == EKeys::SpaceBar)
    {
        // Stop the spacebar from being used as a navigation key.
        return EUINavigationAction::Invalid;
    }

    return FNavigationConfig::GetNavigationActionForKey(InKey);
}
```

In your GameInstance, you set the navigation config:

```cpp
void UMyGameInstance::Init()
{
    Super::Init();

    FSlateApplication::Get().SetNavigationConfig(MakeShared<FAnankeNavigationConfig>());
}

void UMyGameInstance::Shutdown()
{
#if WITH_EDITOR
    // Restore the default navigation config.
    FSlateApplication::Get().SetNavigationConfig(MakeShared<FNavigationConfig>());
#endif

    Super::Shutdown();
}
```

<blockquote style="border-left: 4px solid red; background: #FFCCCB;"><b>IMPORTANT:</b> Don't forget to set your GameInstance class in the project settings as well (Project Settings -> Maps & Modes -> Game Instance -> Game Instance Class)</blockquote>

See this [UE community post](https://dev.epicgames.com/community/snippets/7Wy/custom-remappable-slate-navigation-config) for more info on setting up a custom NavigationConfig.

