---
permalink: /posts/2024-08-26-PSAHoudiniEngineVertexColors/
layout: single
title:  "PSA: Houdini Engine Applies Gamma Correction to Vertex Colors"
date:   2024-08-26 19:46:14 -0800
show_date: true
classes: wide
sidebar:
  nav: "docs"
categories: houdini
author_profile: true
---

I was recently working on some improvements to my river generator, and I ran into an interesting issue: somehow, my flowmaps had experienced some kind of regression as they were no longer following the path of my spline. At first, I thought that I must have introduced a bug into the VEX code responsible for flowmap generation in my HDA, but after investigating, I could not find any issue with it. 



The root cause turned out to be that Houdini Engine applies [gamma correction](https://en.wikipedia.org/wiki/Gamma_correction) to the vertex colors of all generated objects. The reason my tool originally was not affected by this was because there was actually a bug in my version of Houdini Engine (2.1.2) where proxy meshes did not have the gamma correction applied. The bug was fixed [here](https://github.com/sideeffects/HoudiniEngineForUnreal/commit/6c0aa4020f700ac922996cbaab5191d12c20a4ca) and so when I updated to 2.1.4, the flowmaps started using the gamma corrected values. 



I'm not totally sure why SideFx is applying gamma correction to vertex colors. As far as I know, it is common practice to pack vertex color data with various raw data values (like masks or flowmaps), and I'm pretty sure it is preferable to keep these values in linear color space. Regardless, the fix on the UE side is actually pretty straightforward: just use the **sRGBToLinear** node in your material graph to undo the gamma correction:

![](/assets/images/flowmap_fix.png) 



Note: It appears that SideFx is [adding an option](https://github.com/sideeffects/HoudiniEngineForUnreal/commit/ac0186f5e3179f64267d3fdaf12ae3e246a0bc3e) to disable gamma correction, but this feature has not made it into a release yet (as of HE 2.2.1).