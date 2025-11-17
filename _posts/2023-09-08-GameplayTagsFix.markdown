---
layout: post
title:  "UE5 - How to Fix C++ Defined Gameplay Tags Disappearing from Project Settings"
date:   2023-09-08 19:46:14 -0800
categories: 
---

# UE5 - How to Fix C++ Defined Gameplay Tags Disappearing from Project Settings

Stop reading if you are not interested in:

- Defining gameplay tags in C++ UE5 plugins.
- Referencing those gameplay tags from the project settings for a different plugin.


A while back I ran into an issue where a gameplay tag defined in one of my C++ plugins and referenced in the CommonUI plugin input settings (Edit -> Project Settings -> CommonUI -> Input Settings -> Actions -> Input Actions) would disappear after closing and reopening the editor.


**Before restarting the editor, the tag UI.Action.Ananke.Escape is referenced in the CommonUI settings without issue:**

![](/assets/images/CommonInuptSettings_Before.png) 


**After restarting the editor, the tag disappears:**

![](/assets/images/CommonInuptSettings_After.png) 


The root cause of this issue seems to have something to do with the order in which the settings objects are loaded into memory when the UE5 editor starts up. What I think is happening is that the editor is loading DefaultInput.ini (and populating a UCommonUIInputSettings object) before it loads my plugin containing the gameplay tag definitions. 

My solution for this was to create a separate module for our gameplay tags files and then to set the LoadingPhase for that module to "PreDefault". This causes the gameplay tags to load early and results in the tags correctly appearing in the CommonUI input settings, even after the editor is restarted.


```json
{
  "Modules": [
    {
      "Name": "AnankeUILoadEarly",
      "Type": "Runtime",
      "LoadingPhase": "PreDefault"
    },
    {
      "Name": "AnankeUIRuntime",
      "Type": "Runtime",
      "LoadingPhase": "Default"
    }
  ]
}
```


