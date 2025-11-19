---
permalink: /posts/2024-10-24-MayaUniformMove/
layout: single
title:  "Maya: How to Uniformly Move Two Objects Towards Each Other"
date:   2024-10-24 19:46:14 -0800
show_date: true
classes: wide
sidebar:
  nav: "docs"
categories: maya
author_profile: true
---

Here's a quick trick for when you have two objects in maya and you want them to move towards each other by exactly the same distance:

![](/assets/images/goal.png) 



First, make sure that the pivot points of the objects you want to move are EXACTLY lined up on the axis you want to move them on. The easiest way to do that is to snap the objects to the grid using [grid snapping](https://help.autodesk.com/view/MAYAUL/2024/ENU/?guid=GUID-E6E866EE-EEE8-4974-A3E7-9AD6ADBB9BCD).

![](/assets/images/cube_cone_pivot.png) !



Next, create a locator (Create -> Locator) and position it EXACTLY between the pivots of the two objects you want to move.

![](/assets/images/create_locator.png) 

![](/assets/images/locator.png) 



Next do this one at a time for each object:

* With the rigging workspace enabled, select the locator and then one of the objects. 
* Select Constrain -> Parent from the top top toolbar

![](/assets/images/rigging_constrain.png) 



Now that you have constraints set up, if you scale the locator (scale it uniformly by dragging from the center), the objects should move towards each other when you scale down and away from each other when you scale up.

![](/assets/images/scale_locator.png) 



A few extra notes:

* This trick works if you want to move one group of objects towards another group of objects as well. Just make sure that the pivot points of the groups are EXACTLY lined up on the axis you want and the locator is EXACTLY between the pivots of the two groups.
* If you set the scale back to 1.0, it will reset the objects to whatever position you had them in when you set up the constraint.
* If you try to freeze transformations on the locator, it will reset the positions of the objects. If you want to pick a new "default" position for the objects, just delete the locator and parent a new one.