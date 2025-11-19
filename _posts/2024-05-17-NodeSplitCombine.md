---
permalink: /posts/2024-05-17-NodeSplitCombine/
layout: single
title:  "Custom Houdini Nodes: Node Combine and Node Split"
date:   2024-05-17 19:46:14 -0800
show_date: true
classes: wide
sidebar:
  nav: "docs"
categories: houdini
author_profile: true
---

Occasionally, you may run into a case where you need to combine two separate node networks into one. I ran into this issue recently where I wanted to output *two* things from a foreach node loop, but only a single output is supported. As it turns out, the solution is quite simple: just use the [Pack](https://www.sidefx.com/docs/houdini/nodes/sop/pack.html) node to pack each of the separate streams, merge the results, then later you can split up and unpack the results.

There's actually an existing node that does exactly that: [RBD Pack](https://www.sidefx.com/docs/houdini/nodes/sop/rbdpack.html). It's totally fine to use this node- however, since it was developed specifically for RBD geometry, it has some extra features that aren't needed.

I decided to create my own custom combine and split nodes that do just the pack/unpack behavior and nothing else.

![](/assets/images/01_top_level.png) 



For the combine node, all you need to do is pack the inputs and add a name attribute so that you can split each piece of packed geo up later:

![](/assets/images/02_node_combine.png) 



For the split node, isolate the left split and the right split, then unpack them:

![](/assets/images/03_node_split.png) 