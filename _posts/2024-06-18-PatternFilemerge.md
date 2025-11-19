---
permalink: /posts/2024-06-18-PatternFilemerge/
layout: single
title:  "Custom Node: Pattern Filemerge"
date:   2024-06-18 19:46:14 -0800
show_date: true
classes: wide
sidebar:
  nav: "docs"
categories: unreal
author_profile: true
---



I recently ran into a situation where I needed to load a bunch of geometry data from multiple files, but the filenames did not match the required pattern for using the [filemerge](https://www.sidefx.com/docs/houdini/nodes/sop/filemerge.html) node. With this node, each file must match the pattern "some_string_SLICE" where SLICE is a unique index. I decided to write my own filemerge node that uses a python [glob](https://docs.python.org/3/library/glob.html) pattern matcher instead.

The node itself is fairly simple: it just scans through files in the specified directory and creates a point attribute for each matching file. Then, it runs a foreach node loop and loads each file.

![](/assets/images/01_pfm_top_level.png) 



Here's the python for matching the files:

```python
import glob
import os

node = hou.pwd()
parent = node.parent()
geo = node.geometry()

root_directory = parent.evalParm('root_dir')
filepattern = parent.evalParm('filepattern')
attribute_name = parent.evalParm('path_attribute_name')

if (root_directory and filepattern and attribute_name):
    geo.addAttrib(hou.attribType.Point, attribute_name, "")
    files = glob.glob(filepattern, root_dir=root_directory, recursive=True)
    
    for filename in files:
        full_path = os.path.join(root_directory, filename)
        new_point = geo.createPoint()
        new_point.setAttribValue(attribute_name, full_path)
```

