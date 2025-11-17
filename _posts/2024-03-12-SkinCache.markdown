---
layout: post
title:  "What Exactly does 'Compute Skin Cache' Do?"
date:   2024-03-12 19:46:14 -0800
categories: unreal
---

In order to enable HWRT (Hardware Ray Tracing) in UE5, a somewhat mysterious setting must be enabled: *Support Compute Skin Cache*. 



If you look for an explanation in the project settings you get this wonderfully opaque discription:

> *"Cannot be disabled while Ray Tracing is enabled as it is then required."*



If you actually look at the comment in the UE5 source code for this config option, you get a bit more information:

```cpp
// inside RendererSettings.h

/**
	"Skin cache allows a compute shader to skin once each vertex, save those results into a new buffer and reuse those calculations when later running the depth, base and velocity passes. This also allows opting into the 'recompute tangents' for skinned mesh instance feature. Disabling will reduce the number of shader permutations required per material. Changing this setting requires restarting the editor."
	*/
```

```cpp
// inside GPUSkinCache.h

/*=============================================================================
	GPUSkinCache.h: Performs skinning on a compute shader into a buffer to avoid vertex buffer skinning.
=============================================================================*/
```





> (inside GPUSkinCache.h)
>
> *"Performs skinning on a compute shader into a buffer to avoid vertex buffer skinning."*



Finally, checking the UE [documentation](https://docs.unrealengine.com/5.0/en-US/skeletal-mesh-rendering-paths-in-unreal-engine/) yields: 

> *"The **Skin Cache** system skins positions and normals / tangents using a **Compute Shader**, and the results are cached in vertex buffers then passed to `GPUSkinPassThroughVertexFactory` — a variation of `LocalVertexFactory` — for rendering."*



None of this is particularly satisfactory without some additional background information.



### Simplified Version

**Skinning** is the process in the graphics pipeline by which vertices are adjusted based on the "bones" of a skeletal mesh and the weights that have been painted for that mesh.  So basically, the skin cache is a low-level feature that helps with computing how your mesh's "skin" interacts with its "skeleton". By default, Ureal Engine's hardware ray tracing system automatically sends all skeletal meshes through the skin cache. 



### Technical Version

**Skinning** is the process in the graphics pipeline by which vertices are adjusted based on the "bones" of a skeletal mesh (which under the hood are just 3x4 or 4x4 matrices) and the weights that have been painted for that mesh. When the skin cache is enabled, skinning can be performed in the compute shader (a shader stage that is used for computing arbitrary information) instead of in the vertex shader.



From what I can tell, the main advantage for performing skinning in the compute shader instead of the vertex shader is that with the vertex shader, skinning is wastefully recomputed multiple times- once for each render pass. So for example, vertex skinning would happen once for the base pass, and then *again* for the depth pass. The skin cache simply facilitates calculating everything once beforehand, and then each render pass can look up that information when it is needed.



In Unreal, if you want to use hardware ray tracing on static objects, but want to turn off ray tracing on skeletal meshes, you can actually do this with the `r.RayTracing.Geometry.SupportSkeletalMeshes` console option.



### Resources

* [polycount - vertex skinning](http://wiki.polycount.com/wiki/Vertex_skinning)
* [UE5 - Skin Cache System](https://docs.unrealengine.com/5.3/en-US/skeletal-mesh-rendering-paths-in-unreal-engine/#skincachesystem)
* [UE5 - Ray Tracing Skin Cache Rendering Requirements](https://docs.unrealengine.com/5.3/en-US/skeletal-mesh-rendering-paths-in-unreal-engine/#raytracingandhairstrandskincacherenderingrequirements)
* [OpenGL - Compute Shader](https://www.khronos.org/opengl/wiki/Compute_Shader)
* [DirectX - Compute Shader](https://learn.microsoft.com/en-us/windows/win32/direct3d11/direct3d-11-advanced-stages-compute-shader)
* [Unreal's Rendering Passes](https://unrealartoptimization.github.io/book/profiling/passes/)
* [Doom Eternal Study - Compute Shader](https://simoncoenen.com/blog/programming/graphics/DoomEternalStudy#gpu-skinning)
* [Wicked Engine - Skinning in a Compute Shader](https://wickedengine.net/2017/09/09/skinning-in-compute-shader/)

