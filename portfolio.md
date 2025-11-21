---
permalink: /portfolio/
layout: single
author_profile: true
title: "Portfolio"
classes: wide-page-layout

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

# Environment

<div style="display: flex; flex-wrap: wrap; margin-bottom: 2em;">
  <div style="flex: 1 1 60%; padding-right: 20px; min-width: 400px;">
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
  <div style="flex: 1 1 40%; padding-left: 20px; min-width: 300px;">
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
