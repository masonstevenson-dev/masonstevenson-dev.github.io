---
permalink: /portfolio/
layout: single
author_profile: true
title: "Tech Art Portfolio - Mason Stevenson"
classes: portfolio-wide

# --- Swiper Assets ---
# You need to load Swiper CSS and JS for the carousel to work.
header:
  overlay_color: "#333"
  overlay_filter: "0.5"
  # Add external links for Swiper Library
  head:
    - tag: <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/swiper@11/swiper-bundle.min.css">
    - tag: <script src="https://cdn.jsdelivr.net/npm/swiper@11/swiper-bundle.min.js"></script>
# ---------------------
---

<h1 style="margin-bottom: 0px; font-size: 250%;">Environment</h1>
<hr style="
    border: none; 
    background-color: grey;
    height: 3px; 
    width: 100%;
    margin-top: 0px;
">

<div style="display: flex; flex-wrap: wrap; margin-top: 2em;">
  <div style="flex: 1 1 60%; padding-right: 40px; min-width: 400px;">
    <div class="swiper swiper_gallery_landscapes" id="swiper_gallery_landscapes">
        <div class="swiper-wrapper">
        <div class="swiper-slide">
            <img src="/assets/images/portfolio/cross_section_zoomed.webp">
        </div>
        <div class="swiper-slide">
            <img src="/assets/images/portfolio/arid.webp">
        </div>
        <div class="swiper-slide">
            <img src="/assets/images/portfolio/lake.webp">
        </div>
        <div class="swiper-slide">
            <img src="/assets/images/portfolio/canyon.webp">
        </div>
        <div class="swiper-slide">
            <img src="/assets/images/portfolio/mountian_lakes_cross_1.webp">
        </div>
        <div class="swiper-slide">
            <img src="/assets/images/portfolio/mountain_lakes_aerial_1.webp">
        </div>
        <div class="swiper-slide">
            <img src="/assets/images/portfolio/mountain_lakes_ground_1.webp">
        </div>
        </div>
        <div class="swiper-button-next"></div>
        <div class="swiper-button-prev"></div>
    </div>
    <div thumbsslider="" style="display: block;" class="swiper swiper_thumbnail swiper_thumbnail_landscapes">
    <div class="swiper-wrapper">
        <div class="swiper-slide swiper-thumbnail-slide">
        <img src="/assets/images/portfolio/cross_section_zoomed.webp">
        </div>
        <div class="swiper-slide swiper-thumbnail-slide">
        <img src="/assets/images/portfolio/arid.webp">
        </div>
        <div class="swiper-slide swiper-thumbnail-slide">
        <img src="/assets/images/portfolio/lake.webp">
        </div>
        <div class="swiper-slide swiper-thumbnail-slide">
        <img src="/assets/images/portfolio/canyon.webp">
        </div>
        <div class="swiper-slide swiper-thumbnail-slide">
        <img src="/assets/images/portfolio/mountian_lakes_cross_1.webp">
        </div>
        <div class="swiper-slide swiper-thumbnail-slide">
        <img src="/assets/images/portfolio/mountain_lakes_aerial_1.webp">
        </div>
        <div class="swiper-slide swiper-thumbnail-slide">
        <img src="/assets/images/portfolio/mountain_lakes_ground_1.webp">
        </div>
    </div>
    </div>
    </div>
  <div style="flex: 1 1 40%; padding-right: 40px; min-width: 300px;">
    <h2 style="margin-top: 0px;">Terrain</h2>
    <p>
    My general workflow when creating terrain to produce heightmaps using Quadspinner Gaea, add additional procedural features such as rivers, paths, and clouds in Houdini, then set up detailed landscape materials in Unreal Engine.
    </p>
    <p>
    I am particularly interested in developing tools and automation to help environment artists create outdoor scenes quickly and efficiently, as well as deep diving on ways to improve engine performance.
    </p>
    <p>
    See the linked blog posts below for more detailed technical breakdowns of various terrain projects.
    </p>
    <a href="/posts/2024-04-06-LandscapeMaterialBlending/" class="btn btn--primary" target="_blank" style="margin-right: 10px;">Blog: Blending Landscape Materials</a>
    <a href="/posts/2024-09-20-LandscapeTilingTechniques/" class="btn btn--primary" target="_blank" style="margin-right: 10px;">Blog: Texture Variation Techniques in Unreal</a>
    <a href="/posts/2024-06-07-HeightmapBlacklevel/" class="btn btn--primary" target="_blank">Blog: Fix Heightmap Blacklevels with Python</a>
  </div>
</div>

<h1 style="margin-top: 2em; margin-bottom: 0em; font-size: 250%;">Editor Tools</h1>
<hr style="
    border: none; 
    background-color: grey;
    height: 3px; 
    width: 100%;
    margin-top: 0px;
">

<div style="display: flex; flex-wrap: wrap; margin-bottom: 2em;">
  <div style="flex: 1 1 40%; padding-right: 40px; min-width: 300px;">
    <h2 style="margin-top: 0px;">Hex Engine</h2>
    <p>
    I developed a custom editor tool for building custom hex map overlays on top of Unreal Engine landscapes. The tool implements common hex math equations in both C++ (with exposed blueprints) and HLSL. The overlay can be activated as a "preview mode" post process material that projects on top of everything in the scene, with a couple masking options provided: height based masking, which reads landscape height data from a render target and masks out pixels with a height value too far away from the landscape, and custom depth buffer masking which masks out objects that have been written into the custom depth buffer.
    </p>
  </div>
  <div style="flex: 1 1 60%; padding-right: 40px; min-width: 400px;">
    <iframe width="560" height="315" src="https://www.youtube.com/embed/U0OrdfuwQKM?si=3l8QZ6-dK9PGpF0T" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen>
    </iframe>
  </div>
</div>
<div style="display: flex; flex-wrap: wrap; margin-bottom: 6em;">
  <div style="flex: 1 1 40%; padding-right: 40px; min-width: 300px;">
    <p>
    The tool also provides a selection mode where users can select hexes directly in the editor viewport. This mode works by peforming line traces against the landscape based on the position of the cursor and then writing selection data into a custom render target where each pixel contains packed hex selection data for each hex in the grid.
    </p>
    <a href="https://github.com/masonstevenson-dev/HexEngine" class="btn btn--primary" target="_blank" style="margin-right: 10px;">GitHub - Hex Engine</a>
  </div>
  <div style="flex: 1 1 60%; padding-right: 40px; min-width: 400px;">
    <iframe width="560" height="315" src="https://www.youtube.com/embed/qFVvcpCfabI?si=AOiYj8CQvU_eDIKc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
  </div>
</div>
<div style="display: flex; flex-wrap: wrap; margin-bottom: 2em;">
  <div style="flex: 1 1 60%; padding-right: 40px; min-width: 400px;">
    <iframe width="560" height="315" src="https://www.youtube.com/embed/Tbzxm4FqhpI?si=VxKzvvQVR0qiVL5U" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
  </div>
  <div style="flex: 1 1 40%; padding-right: 40px; min-width: 300px;">
    <h2 style="margin-top: 0px;">Automation Graph</h2>
    <p>
    I built a custom graph editor for Unreal Engine that allows you to automate the process of cooking multiple Houdini assets at once. The graph system also supports triggering non-houdini specific actions, such as calling UE console commands.
    </p>
    <p>
    The graph editor is written entirely in C++ and packaged as an Unreal Engine uplugin. Visit the "Houdini Build Sequence Graph" github link below to see the code and additional documentation.
    </p>
  </div>
</div>
<div style="display: flex; flex-wrap: wrap; margin-bottom: 2em;">
  <div style="flex: 1 1 60%; padding-right: 40px; min-width: 400px;">
    <img src="/assets/images/portfolio/AutomationGraph.png">
  </div>
  <div style="flex: 1 1 40%; padding-right: 40px; min-width: 300px;">
    <p>
    I later generalized the Houdini graph code into a framework for Unreal Engine automation. This framework does not have any dependency on Houdini Engine and adds additional functionality like trigger nodes that can cause a graph to execute on editor startup, and a node for executing unit tests.
    </p>
    <p>
    The graph editor is written entirely in C++ and packaged as an Unreal Engine uplugin. Visit the "Automation Graph" github link below to see the code and additional documentation.
    </p>
    <a href="https://www.youtube.com/watch?v=Tbzxm4FqhpI" class="btn btn--primary" target="_blank" style="margin-right: 10px;">Overview Video - Houdini Build Sequence Graph</a>
    <a href="https://github.com/masonstevenson-dev/EnhancedHoudiniEngine" class="btn btn--primary" target="_blank" style="margin-right: 10px;">GitHub - Houdini  Build Sequence Graph</a>
    <a href="https://github.com/masonstevenson-dev/AutomationGraph" class="btn btn--primary" target="_blank">GitHub - Automation Graph</a>
  </div>
</div>

<h1 style="margin-top: 2em; margin-bottom: 0em; font-size: 250%;">Gameplay Systems</h1>
<hr style="
    border: none; 
    background-color: grey;
    height: 3px; 
    width: 100%;
    margin-top: 1px;
">

<div style="display: flex; flex-wrap: wrap; margin-bottom: 2em;">
  <div style="flex: 1 1 40%; padding-right: 40px; min-width: 300px;">
    <h2 style="margin-top: 0px;">Mantle ECS</h2>
    <p>
    Back in 2023, I built a Entity Component System (ECS) for Unreal Engine because I wanted to better understand how the Mass Entity ECS system provided by Epic works under the hood. This ECS takes much inspiration from the Mass Entity, but was built from scratch with a more simplified API and a focus on exploring how ECS systems can be integrated with other gameplay systems like player perception or ability systems.
    </p>
    <p>
    Something still bugged me: I was interested in capturing some of the organizational benefits of ECS archetecture, but felt that this system still had too much bespoke memory management and complex internal code. More recently, I've decided to do a full re-write with a focus on using the standard Unreal framework as much as possible.
    </p>
    <p>
    To see the code and additional documation, see the github links below.
    </p>
    <a href="https://github.com/masonstevenson-dev/Mantle" class="btn btn--primary" target="_blank" style="margin-right: 10px;">GitHub - Mantle ECS</a>
    <a href="https://github.com/masonstevenson-dev/Mantle2" class="btn btn--primary" target="_blank" style="margin-right: 10px;">GitHub - Mantle2</a>
  </div>
  <div style="flex: 1 1 60%; padding-right: 40px; min-width: 400px;">
    <img src="/assets/images/portfolio/MantleEcsArchitecture.png">
  </div>
</div>

<h1 style="margin-top: 2em; margin-bottom: 0em; font-size: 250%;">Procedural</h1>
<hr style="
    border: none; 
    background-color: grey;
    height: 3px; 
    width: 100%;
    margin-top: 0px;
">

<div style="display: flex; flex-wrap: wrap; margin-bottom: 2em;">
  <div style="flex: 1 1 40%; padding-right: 40px; min-width: 300px;">
    <h2 style="margin-top: 0px;">PCG: Extract Collision Boxes</h2>
    <p>
    Here is a PCG tool I created for scattering mushrooms across a surface. Points are generated on the target mesh and then culled using a number of different strategies:
    </p>
    <ul>
      <li>Placement of the point relative to the height of target mesh</li>
      <li>How much the direction a point is facing deviates from the up vector</li>
      <li>Random pruning</li>
      <li>Overlap reduction</li>
      <li>Intersection of mushroom caps with target mesh (more on this below)</li>
    </ul>
  </div>
  <div style="flex: 1 1 60%; padding-right: 40px; min-width: 400px;">
    <iframe width="560" height="315" src="https://www.youtube.com/embed/xRJ-VotlPMc?si=OJd7xCSye4UDA6Gd" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
  </div>
</div>

<div style="display: flex; flex-wrap: wrap; margin-top: 0em; margin-bottom: 6em;">
  <div style="flex: 1 1 40%; padding-right: 40px; min-width: 300px;">
    <p>
    One interesting problem I ran into was that occasionally the mushrooms would clip into the tree stump mesh. I needed a way to detect intersections of the mushroom caps with the tree stump, while ignoring the mushroom stalks (these are okay to intersect). In this case, doing a simple bounding box detection is not enough.
    </p>
    <p>
    The solution I came up with was to add special collision boxes to the mushroom meshes representing the mushroom caps. I then created a new custom PCG node called ExtractCollisionBoxes that allows you to generate point data from a collision box on a mesh. You can tag the collision box with an id to grab a specific one, otherwise it will just grab the first box it finds. These extracted points can then be pruned based on if they intersect with points sampled from the target mesh. My custom node is written entirely in native C++.
    </p>
    <a href="https://github.com/masonstevenson-dev/pcg-extension" class="btn btn--primary" target="_blank" style="margin-right: 10px;">GitHub - PCG Tools</a>
    <a href="/posts/2024-11-17-PCGDifferenceNode/" class="btn btn--primary" target="_blank" style="margin-right: 10px;">Blog: PCG Difference Node Explained</a>
  </div>
  <div style="flex: 1 1 60%; padding-right: 40px; min-width: 400px;">
    <div class="swiper swiper_gallery_pcgmushrooms" id="swiper_gallery_pcgmushrooms">
      <div class="swiper-wrapper">
      <div class="swiper-slide">
          <img src="/assets/images/portfolio/extract_collision_boxes_1.png">
          <div class="swiper-top-right-caption">Without Collision Box Detection</div>
      </div>
      <div class="swiper-slide">
          <img src="/assets/images/portfolio/custom_collision_box.png">
          <div class="swiper-top-right-caption">Add Custom Collision Box</div>
      </div>
      <div class="swiper-slide">
          <img src="/assets/images/portfolio/extract_collision_boxes_2.png">
          <div class="swiper-top-right-caption">Collision Boxes Visualized</div>
      </div>
      <div class="swiper-slide">
          <img src="/assets/images/portfolio/extract_collision_boxes_3.png">
          <div class="swiper-top-right-caption">Colliding Meshes Culled</div>
      </div>
      </div>
      <div class="swiper-button-next"></div>
      <div class="swiper-button-prev"></div>
    </div>
    <div thumbsslider="" style="display: block;" class="swiper swiper_thumbnail swiper_thumbnail_pcgmushrooms">
      <div class="swiper-wrapper">
        <div class="swiper-slide swiper-thumbnail-slide">
        <img src="/assets/images/portfolio/extract_collision_boxes_1.png">
        </div>
        <div class="swiper-slide swiper-thumbnail-slide">
        <img src="/assets/images/portfolio/custom_collision_box.png">
        </div>
        <div class="swiper-slide swiper-thumbnail-slide">
        <img src="/assets/images/portfolio/extract_collision_boxes_2.png">
        </div>
        <div class="swiper-slide swiper-thumbnail-slide">
        <img src="/assets/images/portfolio/extract_collision_boxes_3.png">
        </div>
      </div>
    </div>
  </div>
</div>

<div style="display: flex; flex-wrap: wrap; margin-bottom: 2em;">
  <div style="flex: 1 1 60%; padding-right: 40px; min-width: 400px;">
    <iframe width="560" height="315" src="https://www.youtube.com/embed/Tbzxm4FqhpI?si=VxKzvvQVR0qiVL5U" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
  </div>
  <div style="flex: 1 1 40%; padding-right: 40px; min-width: 300px;">
    <h2 style="margin-top: 0px;">River Generator</h2>
    <p>
    I created a river generator HDA that takes an Unreal Engine landscape spline as input and then generates a river mesh based on the path of the spline and the width of each spline segment. Flowmap data for scrolling the water material along the path of the spline is computed in Houdini and stored in the vertex colors of the mesh.
    </p>
    <p>
    One challenge I ran into was that the SideFX labs flowmap generator produces colors in UV space, which leads to artifacts at each bend in the spline. I rewrote the VEX code for computing the flowmaps to output world-space colors, and was able to get a much cleaner result.
    </p>
  </div>
</div>
<div style="display: flex; flex-wrap: wrap; margin-bottom: 6em;">
  <div style="flex: 1 1 60%; padding-right: 40px; min-width: 400px;">
    <a href="/assets/images/portfolio/river_mat_3.png" target="_blank">
      <img src="/assets/images/portfolio/river_mat_3.png">
    </a>
  </div>
  <div style="flex: 1 1 40%; padding-right: 40px; min-width: 300px;">
    <p>
    For the river material, I created a simplified version of the water material included in the <a href="https://dev.epicgames.com/documentation/en-us/unreal-engine/water-system-in-unreal-engine" target="_blank">UE Water System Plugin</a>.
    </p>
    <p>
    One thing I would like to improve in the future is the realism of the ripple effect. With the stock UE "flowmaps" node, I'm still getting a bit of noticeable flicker as it blends between the normalmap and the time-adjusted normalmap. I'd like to explore more techniques for improving this, such as <a href="https://www.youtube.com/watch?v=VHet3I4u614&t=9120s" target="_blank">this technique</a> from Prismatica that adds a third blend so that at any given time, one texture sample is being blended in, one is being blended out, and one is steady.
    </p>
    <p>
    To see the code and additional documation, see the github links below.
    </p>
    <a href="/posts/2024-05-24-Flowmaps/" class="btn btn--primary" target="_blank" style="margin-right: 10px;">Blog - Understanding TexCoord & Flowmaps</a>
  </div>
</div>

<div style="display: flex; flex-wrap: wrap; margin-bottom: 2em;">
  <div style="flex: 1 1 40%; padding-right: 40px; min-width: 300px;">
    <h2 style="margin-top: 0px;">Fence Generator</h2>
    <p>
    I created this fence generator HDA that assembles prefabricated fence sections and runs a stagger computation that splits up the sections into smaller subsections based on how sloped the landscape is.
    </p>
    <p>
    Fence pickets and posts are exported from Houdini as <a href="https://dev.epicgames.com/documentation/en-us/unreal-engine/instanced-static-mesh-component-in-unreal-engine#instancedstaticmesh" target="_blank">Instanced Static Meshes</a>, and additionally the posts are rotated randomly for some visual breakup.
    </p>
  </div>
  <div style="flex: 1 1 60%; padding-right: 40px; min-width: 400px;">
    <iframe width="560" height="315" src="https://www.youtube.com/embed/wek17gXQEKQ?si=xoesGpHwY6xpe-04" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
  </div>
</div>
<div style="display: flex; flex-wrap: wrap; margin-bottom: 2em;">
  <div style="flex: 1 1 40%; padding-right: 40px; min-width: 300px;">
    <p>
    My original goal was to create a generalized fence generator that could support many different fence types. As I worked on the project, I realized that making a lightwight tool (in terms of number of configurable parameters) is feels more practical since most users of the tool are probably mostly concerned with making the "big decisions" as opposed to having dozens of small tweakable parameters.
    </p>
    <p>
    One thing I would like to improve in the future is supporting more fine-grained control at the end of the spline. Due to the fact that this tool does not allow stretching, the post placed at the last point on the spline cannot easily be positioned exactly where you want it. In the future, I would like to find a way that both ends can be positioned exactly.
    </p>
    <a href="/posts/2024-07-19-FencegenProjectBreakdown/" class="btn btn--primary" target="_blank" style="margin-right: 10px;">Blog - Fence Project Breakdown</a>
  </div>
  <div style="flex: 1 1 60%; padding-right: 40px; min-width: 400px;">
    <img src="/assets/images/top_level_controls.png">
  </div>
</div>

<h1 style="margin-top: 2em; margin-bottom: 0em; font-size: 250%;">Pipeline</h1>
<hr style="
    border: none; 
    background-color: grey;
    height: 3px; 
    width: 100%;
    margin-top: 1px;
">

<div style="display: flex; flex-wrap: wrap; margin-bottom: 2em;">
  <div style="flex: 1 1 60%; padding-right: 40px; min-width: 400px;">
    <iframe width="560" height="315" src="https://www.youtube.com/embed/JiLfNQtbHPM?si=worw9PLYul7fEmLa" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
  </div>
  <div style="flex: 1 1 40%; padding-right: 40px; min-width: 300px;">
    <h2 style="margin-top: 0px;">JSON Asset Definition Library</h2>
    <p>
    I created a Python library for saving Houdini parameter configurations out to a JSON file. The initial motivation behind this project was due to the fact that I wanted to create a TOPnet that could generate variants of some particular geometry based on a file definition. I first tried using the built-in Houdini parameter <a href="https://www.sidefx.com/docs/houdini/network/parms.html#presets" target="_blank">preset</a> feature, but had trouble accessing the preset from inside the TOPnet.
    </p>
    <p>
    An added benefit to going the JSON route is that the file is human readable and the common data format makes it ideal for integrating into larger pipelines.
    </p>
  </div>
</div>
<div style="display: flex; flex-wrap: wrap; margin-bottom: 2em;">
  <div style="flex: 1 1 60%; padding-right: 40px; min-width: 400px;">
    <iframe width="560" height="315" src="https://www.youtube.com/embed/hjVLm2Y9yRg?si=r6PJLBwX12autUah" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
  </div>
  <div style="flex: 1 1 40%; padding-right: 40px; min-width: 300px;">
    <p>
    I later added support for multiparms as well. I used this multiparm functionality in a headstone generator I created to define an arbitrary number of cracks for each headstone asset definition.
    </p>
  </div>
</div>

<script>
  var st_pcgmushrooms = new Swiper(".swiper_thumbnail_pcgmushrooms", {
    spaceBetween: 0,
    slidesPerView: 4,
    freeMode: true,
    watchSlidesProgress: true,
  });
  var sg_pcgmushrooms = new Swiper(".swiper_gallery_pcgmushrooms", {
    spaceBetween: 10,
    navigation: {
    nextEl: ".swiper-button-next",
    prevEl: ".swiper-button-prev",
    },
    thumbs: {
        swiper: st_pcgmushrooms,
    },
  });
  // landscapes
  var st_landscapes = new Swiper(".swiper_thumbnail_landscapes", {
    spaceBetween: 0,
    slidesPerView: 4,
    freeMode: true,
    watchSlidesProgress: true,
  });
  var sg_landscapes = new Swiper(".swiper_gallery_landscapes", {
    spaceBetween: 10,
    navigation: {
    nextEl: ".swiper-button-next",
    prevEl: ".swiper-button-prev",
    },
    thumbs: {
        swiper: st_landscapes,
    },
    on: {
        slideChange: function () {
            var activeIndex = sg_landscapes.activeIndex;
            st_landscapes.slideTo(activeIndex);
        } 
    },
  });
  // rivermat
  var sg_rivermat = new Swiper(".swiper_gallery_rivermat", {
    zoom: false,
    spaceBetween: 10,
    pagination: {
        el: ".swiper-pagination",
        clickable: false,
    },
  });
  // unhide all swiper thumbnails.
  var elements = document.querySelectorAll('.swiper_thumbnail');
  elements.forEach(function(element) {
    element.style.display = 'block';
  });
</script>
