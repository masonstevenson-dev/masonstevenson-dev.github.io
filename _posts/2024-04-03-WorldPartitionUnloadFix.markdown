---
permalink: /posts/2024-04-03-WorldPartitionUnloadFix/
layout: single
title:  "World Partition Regions Won't Unload: How to Fix"
date:   2024-04-03 19:46:14 -0800
classes: wide
sidebar:
  nav: "docs"
categories: 
---

I recently ran into this bug in UE 5.3 where unreal refused to unload my landscape that I had imported via a heightmap from Gaea. This is especially strange because the landscape will appear as loaded in the editor even when they are marked as unloaded in the world partition minimap. Loading and unloading the regions manually from there does not help.

![](/assets/images/01_problem.png) 


I was not able to determine what causes this to happen, but I did find a workaround. In order to get your landscape to unload in the editor, first go to the World Settings tab and uncheck **enable streaming**:

![](/assets/images/02_enable_streaming.png) 


Then save your level and restart the editor. Finally, go back to the World Settings tab and check **enable streaming**. The editor may lock up for a few seconds, but once it is finished re-enabling streaming, your landscape should be properly unloaded. At this point, you should be able to manually load and unload individual regions from the world partition minimap.

![](/assets/images/03_fixed.png)



