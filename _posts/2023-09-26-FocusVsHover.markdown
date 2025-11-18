---
permalink: /posts/2023-09-26-FocusVsHover/
layout: single
title:  "UE Widget Focus vs Hover States"
date:   2023-09-26 19:46:14 -0800
show_date: true
classes: wide
sidebar:
  nav: "docs"
categories: 
author_profile: true
---

One major confusion I've had while working with Unreal Engine widgets is how UE handles widget *focus* vs widget *hover* states. If you've spent any time working with the UI framework, you may have experienced something along the lines of the following:

You have an on-screen button menu that you want to display to the player. You want the currently selected button to light up, so you set the button's "hover" style to be a different color and then tell your widget to focus on the button. .... aaaand all you get is this pathetic blue outline: 

![](/assets/images/focus_orange.png)

<blockquote style="border-left: 4px solid darkgrey; background: #dfe2e5;"> <b>NOTE:</b> If you don't even get the blue outline, you probably have your widget render focus rule turned off. Go to Edit->Project Settings->Engine->User Interface and set "Render Focus Rule" to "Always".</blockquote>

This is because your widget is currently *focused* but not *hovered*. Focus has to do with what widget the player has currently navigated to (using either the keyboard arrow keys or the gamepad thumbstick/d-pad). Hover, on the other hand, is a special state meaning the mouse cursor is currently on top of the widget. Annoyingly, UE Button widgets have built-in styling for when the button is hovered but nothing for handling if the widget is focused:

![](/assets/images/style_options.png)

## But it Works in CommonUI!
Luckily, Epic has provided an extension of their UI framework in the form of the CommonUI plugin. If you've used it, you may have noticed that your buttons magically light up with the proper hover state when you navigate to them... with a gamepad that is. With a keyboard, it still does not behave as expected. In fact, this even caused some confusion for the presenter in Epic's [Introduction to CommonUI](https://www.youtube.com/watch?v=TTB5y-03SnE&t=5222s) presentation from an *Inside Unreal* stream last year.  

### What is going on here?
As it turns out, if you have CommonUI enabled and you are using a gamepad, CommonUI will actually SILENTLY MOVE THE MOUSE CURSOR to the center of whatever widget you have focused.

From *CommonAnalogCursor.cpp*:

```cpp
if (IsUsingGamepad() && IsGameViewportInFocusPathWithoutCapture())
{
    // ...
    
    TSharedPtr<SWidget> CursorTarget = SlateUser->GetFocusedWidget();
    
    // ...
    
    TargetGeometry = CursorTarget->GetCachedGeometry();
    
    // ...
    
    if (TargetGeometry.GetLocalSize().SizeSquared() > SMALL_NUMBER)
	{
        bHasValidCursorTarget = true;
        const FVector2D AbsoluteWidgetCenter = TargetGeometry.GetAbsolutePositionAtCoordinates(
            FVector2D(0.5f, 0.5f)
        );
        SlateUser->SetCursorPosition(AbsoluteWidgetCenter);
    }
    
    // ...
}   
```

CommonUI implements no such cursor shenanigans for players using m+kb, and thus the problem still persists.

## Potential Solutions
Here are a few workarounds you can try:

### Use CommonButton OnFocused
UCommonButtonBase has OnFocusReceived and OnFocusLost events that you can bind to. You could create your own custom button class that sets the style when the button is focused and then restores the old style when focus is lost. 

#### (optional) Force the hover state from Slate
Instead of swapping out styles on focus, another option is to define a custom widget that allows for forcing the hover state. For example, you could create a custom SButton class that exposes the internal SetHover() fn so that it can be used by its parent UCommonButton. Then, when OnFocusReceived is triggered, you can force the button to be "hovered" as well.

### Do what commonUI does, but for m+kb
Another option would be to add custom functionality for keyboard navigation that snaps the cursor to the center of the currently focused widget in the same way that CommonUI does it for gamepad.

