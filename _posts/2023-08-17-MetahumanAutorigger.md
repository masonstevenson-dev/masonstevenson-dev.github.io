---
permalink: /posts/2023-08-17-MetahumanAutorigger/
layout: single
title:  "Setting up an Autorigger for Metahumans"
date:   2023-08-17 19:46:14 -0800
show_date: true
classes: wide
sidebar:
  nav: "docs"
categories: unreal
author_profile: true
---

Here's a quick project I threw together to learn how writing plugins for Maya works:

<iframe width="560" height="315" src="https://www.youtube.com/embed/pimnXcWN7tg?si=3wHBieNgDX48KeP-" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>



I wrote this plugin using a mix of C++ and MEL. When my plugin is initialized with the initializePlugin fn (see [here](https://help.autodesk.com/view/MAYAUL/2024/ENU/?guid=Maya_SDK_A_First_Plugin_cpp_Hello_World_Explained_html) for more info), it loads my custom MEL scripts using the [source](https://help.autodesk.com/cloudhelp/2023/ENU/Maya-Tech-Docs/Commands/source.html) MEL command. Additionally, it calls [registerUI](https://help.autodesk.com/cloudhelp/2022/ENU/Maya-SDK/cpp_ref/class_m_fn_plugin.html#a7c72455a763ee8c34481f3fe786659ae) to and passes in the creation and deletion MEL scripts.



The IK rig scripts themselves are a modified version of the ones supplied in this video from yt channel Maya Expert:

<iframe width="560" height="315" src="https://www.youtube.com/embed/iHyA-CJbZdA?si=fBwrPu3L9egJgOfA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>



I was able to clean up the scripts and separate them out into individual commands that could be tied to my UI buttons. 



Overall, I found the process of plugin development for maya to be fairly smooth. The documentation is not incrededible, but is adequate for getting the job done. By far the most difficult part was building the UI using MEL. The syntax is pretty archaic and it takes a surprising amount of code to even get a simple panel with buttons on it. If I do any more Maya plugin development in the future, I may look into the python API instead for the UI.