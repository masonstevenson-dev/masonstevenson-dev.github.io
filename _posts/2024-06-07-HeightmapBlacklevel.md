---
permalink: /posts/2024-06-07-HeightmapBlacklevel/
layout: single
title:  "How to adjust heightmap black levels with Python"
date:   2024-06-07 19:46:14 -0800
show_date: true
classes: wide
sidebar:
  nav: "docs"
categories: gaea
author_profile: true
---

Recently when I was playing around with exporting heightmap data from Gaea to Unreal, I spent some time solving a small problem with my landscape height. I wanted the lowest point on the map to have a z position of 0.0, however I found that there was no nice way to do this inside Gaea. Now, it isn't strictly *necessary* to zero out the lowest point on your terrain this way, but I wanted to do this since it gives the z position 0.0 some tangible meaning instead of it being a completely arbitrary location in your game world. I tried a couple different approaches for fixing this issue, but ultimately just ended up writing a short python script.



## Gaea Technique

As I mentioned, Gaea does not have a nice way of adjusting the black point of the heightmap image. The best technique I could find was in [this youtube tutorial](https://www.youtube.com/watch?v=HMv5zmPyl2s&t=595s), where they autolevel the terrain so that the heightmap values are remapped from 0 to H (where H is the max height). Then they use a clamp max to push the height back down. What I don't like about this approach is that if you are already happy with your terrain, you are needlessly mutating that good terrain layout, just to get a blacklevel adjustment.



## Photoshop Technique

The next thing I decided to try was photoshop.  The process for zeroing out the darkest pixels is as follows:

1) Go to Image > Adjustments > Threshold
2) Drag the slider down until the entirety of your heightmap image disappears. Then drag the slider 1 value forwards so that the darkest pixels are visible.
3) Make note of this threshold value and then go to Image > Adjustments > Levels
4) Set the black level (the value of the lefthand side) to your threshold value.



Unfortunately, something about the Photoshop export process was corrupting my heightmap image. When I tried importing it to unreal, I got warnings that the imported image was not greyscale. I tried altering some of the .png export settings to ensure that I was exporting an 8bit greyscale png, but was not able to determine what photoshop was doing to the file that was causing Unreal to detect an issue.



## Python

Finally, I gave up and just decided to write a script to do this for me. It turns out that editing the pixels of a png directly with python is surprisingly easy. Here's the code:

```python
import os
from PIL import Image

def process_single_image(path, is_relative_path=True):
    if (is_relative_path):
        path = os.path.relpath(path)
    
    with Image.open(path) as image:
        width, height = image.size

        count = 0
        lowest = 65535 # max value
        pixels = image.getdata()

        for pixel in pixels:
            if pixel < lowest:
                lowest = pixel

        new_pixel_data = []

        for index in range(len(pixels)):
            new_pixel_data.append(pixels[index] - lowest)

        image.putdata(new_pixel_data)

        root, ext = os.path.splitext(path)

        output_path = root + '_ZEROED' + ext
        image.save(output_path)
```

