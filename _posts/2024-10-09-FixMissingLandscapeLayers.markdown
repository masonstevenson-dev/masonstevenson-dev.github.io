---
permalink: /posts/2024-10-09-FixMissingLandscapeLayers/
layout: single
title:  "How to Fix: Unreal Landscape Material Layers Not Showing Up"
date:   2024-10-09 19:46:14 -0800
show_date: true
classes: wide
sidebar:
  nav: "docs"
categories: 
---

When using the Unreal Engine [landscape material system](https://dev.epicgames.com/documentation/en-us/unreal-engine/landscape-materials-in-unreal-engine), each landscape layer must be associated with a corresponding "LayerInfo" data asset (you can read more about this in my previous blog post, [Blending Landscape Materials in Unreal Engine](https://www.exportgeometry.com/blog/unreal-landscape-material-blending)). However, if you have ever tried to add a new Landscape Layer Sample node to an existing landscape material, you may have noticed that sometimes the newly created layer does not actually show up in the list of available layers on the landscape mode paint tab.


**Here I add a new layer, CanyonMask:**

![](/assets/images/new_landscape_layer.png) 


**I go to the list of layers in order to link my new layer with a LayerInfo asset, and the layer is missing:**

![](/assets/images/not_referenced_in_paint.png) 


## How to Fix

If you encounter this issue, you can fix it using the following workaround. 


Make sure you have your landscape mode paint menu OPEN. Then, go to the landscape layer sample node in your material graph and set the preview weight to 1. Be sure to apply your change by pressing the "Apply" button at the top of the material graph editor.

![](/assets/images/set_preview_weight.png)  


You should see your new layer referenced in the paint menu and from here you can link a new LayerInfo asset as normal:

![](/assets/images/added.png) 


(Optional) You can now also revert the preview weight on your landscape layer sample node back to 0.


## Other Possible Workarounds

If the fix above does not work, you can also try:

1) Saving and reloading your map.
2) Removing the landscape material from your landscape and re-adding it.

