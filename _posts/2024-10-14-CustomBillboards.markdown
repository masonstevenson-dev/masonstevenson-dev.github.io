---
layout: post
title:  "Unreal Engine: How to Set Up Custom Actor Billboards From C++"
date:   2024-10-14 19:46:14 -0800
categories: 
---

# Unreal Engine: How to Set Up Custom Actor Billboards From C++



Ever wonder how to set a custom actor billboard from C++? Here's a step by step guide.

![](/assets/images/custom_billboard_halfsize.png) 


First off, if you are planning on adding this to a plugin class, make sure you have CanContainContent set to true in your .uplugin file:

```json
{
  "FileVersion": 3,
  "Version": 1,
  "VersionName": "1.0",
  "EngineVersion": "5.4.0",
  "FriendlyName": "Blog Examples",
  "Description": "Example stuff for the ExportGeometry blog.",
  "Category": "ExGeo",
  "CreatedBy": "mason stevenson",
  "CreatedByURL": "http://exportgeometry.com",
  "DocsURL": "",
  "MarketplaceURL": "",
  "SupportURL": "",
  "EnabledByDefault": true,
  "CanContainContent": true,
  "IsBetaVersion": true,
  "Modules": [
    {
      "Name": "BlogExamplesRuntime",
      "Type": "Runtime",
      "LoadingPhase": "Default"
    }
  ],
  "Plugins": []
}
```


Next, add the image you want to use as a texture to either your project or your plugin. Note that your plugin content folder will only show up if you have set CanContainContent in your uplugin file as outlined above.

![](/assets/images/add_texture.png) 


Now, back in C++, you can set up your actor with a BillboardComponent. If you intend for your actor to be a utility/manager class, you can just have your actor extend **AInfo**:

```c++
#pragma once

#include "TestActor.generated.h"

UCLASS(Blueprintable)
class BLOGEXAMPLESRUNTIME_API ATestActor : public AInfo
{
	GENERATED_BODY()

public:
	ATestActor(const FObjectInitializer& Initializer);
};
```


In your actor's constructor, get the billboard component (with `GetSpriteComponent()` if you are using AInfo) and pass it a Texture2D object that references your custom image:

```c++
#include "Actor/TestActor.h"

#include "Components/BillboardComponent.h"

ATestActor::ATestActor(const FObjectInitializer& Initializer): Super(Initializer)
{
#if WITH_EDITORONLY_DATA
	if (!IsRunningCommandlet() && (GetSpriteComponent() != NULL))
	{
		GetSpriteComponent()->SetSprite(LoadObject<UTexture2D>(
            nullptr, TEXT("/BlogExamples/Textures/custom_billboard_128.custom_billboard_128")));
	}
#endif
}
```


