---
permalink: /posts/2024-01-29-HoudiniJsonLoader/
layout: single
title:  "Creating a JSON Driven Topnet in Houdini"
date:   2024-01-29 19:46:14 -0800
show_date: true
classes: wide
sidebar:
  nav: "docs"
categories: houdini
author_profile: true
---

Last week I put together a project as a proof of concept for how to set up data driven [PDG]() networks in Houdini. The general goal was to create a tool in Houdini that lets artists quickly generate a bunch of different variants of a particular object, with a simple interface for saving and loading the parameters used to generate the object. Then, once all those variants are saved to a file, the system needed to be capable of ingesting that file and producing work items for generating each individual piece of geometry. I managed to get a basic end-to-end system up and running fairly painlessly. Here are the results:

<iframe width="560" height="315" src="https://www.youtube.com/embed/SSujIKtIvxA?si=rMaAr-U5QUabyJWI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>



## Detailed Breakdown

#### A Quick Note on Why I Opted for JSON

Houdini actually has an existing [presets system](https://www.sidefx.com/docs/houdini/ref/windows/savepreset.html). I played around with just using this instead of loading from JSON, but abandoned this idea due to issues with the dynamically generated topnet nodes not knowing where to look for the preset file. It's possible there is still a way to set this up, but I decided to go for loading a JSON file instead which I knew would work. Furthermore, I think JSON is a little more robust as you could hypothetically use this system in a much larger asset pipeline where the asset definition data comes from some source that is external to Houdini.



#### Breakdown

I opted to keep the tool itself as simple as possible. All it does is create a cube with a scale and a color:

![](/assets/images/01_box_generator.png) 



In the "Manual Controls" section, you can specify the path to a JSON file formatted like so:

```json
{
    "schema_version": 1,
    "asset_definitions": [
        {
            "name": "bigRedBox",
            "center": [
                0,
                0,
                0
            ],
            "bg_scale": 5,
            "bg_colorr": 1,
            "bg_colorg": 0,
            "bg_colorb": 0
        },
        {
            "name": "smallGreenBox",
            "center": [
                0,
                1,
                0
            ],
            "bg_scale": 2,
            "bg_colorr": 0,
            "bg_colorg": 1,
            "bg_colorb": 0,
            "bg_bad_param": 9
        }
    ]
}
```

The main syntax constraint on the JSON file is that anything you want to populate a parameter field with has to start with "bg\_" and must correspond exactly to the name of the parameter in Houdini. So in this case, my scale and color fields were named "bg_scale" and "bg_color", and thus the valid JSON keys are:

* bg_scale
* bg_colorr
* bg_colorg
* bg_colorb



Anything that does *not* start with "bg\_" (like the "center" key I added) is completely ignored. Anything that starts with "bg\_" but does not correspond to a parameter in the box generator tool is also ignored (with a warning, if logging is turned on).



The preset menu is dynamically generated when the node cooks by using the [menu script](https://www.sidefx.com/docs/houdini/hom/locations.html#parameter_menu_scripts) option on the ordered menu param. One slightly odd thing about how the menu script works is that even if you pass in the menu token as an integer, it will be implicitly cast to string. In many cases, you can pretty much ignore the tokens completely and just reference the menu items by index using `node.evalParm('your_menu')`.

![](/assets/images/02_menu_script.png) 



Another interesting thing to mention is that you can cache data on the node itself by using the [cachedUserDataDict](https://www.sidefx.com/docs/houdini/hom/nodeuserdata.html). I set up my scripts to lazy-load the json data, i.e. the node will load the data from the JSON file the first time `get_cached_json_data()` is called, and on subsequent calls it will fetch this data from the cache. When the user presses "Reload File", the cache is wiped and then the file is re-read to pull in any updated values.



When the topnet is running, the box generator controls are even more simple. All it needs to generate the geo is the JSON file path and the index of which object to generate:

![](/assets/images/03_pdg_controls.png) 

I originally wanted the topnet to use the same dynamically generated preset menu that you use in manual mode, but I found that there are a lot of weird interactions that arise when you have params with custom scripts attached to them that are dynamically generated in the HDA Processor node.



Inside the box generator, I just have a small python node to pull in the asset data from the JSON file and then populate some detail attributes. If manual mode is enabled, the VEX script below it will override the values from the JSON file with values from the parent node's parameters instead.

<img src="/assets/images/04_inside_bg.png" style="zoom: 50%;" /> 

```python
# inside pdg_load_asset_defs
def main():
  node = hou.pwd()
  generator_node = node.parent()

  if (not generator_node.hm().is_pdg_enabled(generator_node)):
    return
  
  json_data = generator_node.hm().load_json_data(generator_node)
  asset_defs = json_data['asset_definitions']
  asset_index = generator_node.parm('pdg_preset_index').eval()
  
  if (asset_index < 0 or asset_index >= len(asset_defs)):
    return
    
  asset_def = asset_defs[asset_index]
  
  geo = node.geometry()
  geo.addAttrib(hou.attribType.Global, 'scale', asset_def['bg_scale'])
  geo.addAttrib(hou.attribType.Global, 'Cd', (asset_def['bg_colorr'], asset_def['bg_colorg'], asset_def['bg_colorb']))

# ------------------------------------------------------------------
main()
```



Inside the topnet, I have a python processor that opens up the JSON file and counts the number of asset definitions inside. This number is all that is needed for the wedge node to split out the correct number of work items (maybe I could have just done this in the python node?).

![](/assets/images/05_python_processor.png) 



Finally, the HDA processor just needs to grab the index of each work item and feed that into the "Preset Index" param to drive the box generator output.

![](/assets/images/06_box_out.png) 



### Full Python Scripts

#### Box Generator Python Module

```python
import hou
import json
import os.path

def log(node, msg):
  if (not node.parm('enable_logging').eval()):
    return
    
  print(msg)
  
def is_pdg_enabled(node):
  return node.parm('pdg_enabled').eval()

def force_cook(node):
  if (is_pdg_enabled(node)):
    return
  
  node.cook(force=True)
  
def load_json_data(node):  
  json_data = {}

  json_file_path = node.evalParm('json_file_path')
  if (os.path.isfile(json_file_path)):
    log(node, 'loading json data from: ' + json_file_path)
  else:
    log(node, 'unable to load json data. path is invalid: ' + json_file_path)
    return
  
  with open(json_file_path, 'r') as json_file:
    json_data = json.load(json_file)
  
  return json_data
        
def get_cached_json_data(node):
  if (is_pdg_enabled(node)):
    return {}

  cached_node_data = node.cachedUserDataDict()
  
  if ('json_data' not in cached_node_data):
    node.setCachedUserData('json_data', node.hm().load_json_data(node))
    cached_node_data = node.cachedUserDataDict()
        
  return cached_node_data.get('json_data', {})
  
def get_preset_index(node):
  preset_name = node.evalParm('preset_name')
  menu_labels = node.parm('preset_menu').menuLabels()
  
  if (preset_name == '' or preset_name not in menu_labels):
    return max(len(menu_labels) - 1, 0)

  # Match the preset index to whatever is currently in the name field, NOT
  # which menu item is actually selected. This fixes an bugs that manifest when
  # the user picks a preset, changes the 'preset_name' text field, then presses
  # the save button.
  return menu_labels.index(preset_name)
  
def update_changed_params(node):
  preset_index = get_preset_index(node)
  
  json_data = get_cached_json_data(node)
  asset_defs = json_data['asset_definitions']

  if (asset_defs is None or preset_index > len(asset_defs)):
    return
  if (preset_index == len(asset_defs)):
    node.parm('changed_params').set(-1)
    return

  current_asset = asset_defs[preset_index]
  
  changed_params = 0
  
  for parm_id, value in current_asset.items():
    if (parm_id.startswith('bg_') and node.parm(parm_id) and node.parm(parm_id).eval() != value):
      changed_params += 1
      
  node.parm('changed_params').set(changed_params)
      
  
def clear_presets(node):
  if (is_pdg_enabled(node)):
    return

  for parm in node.parms():
    if (parm.name().startswith('bg_')):
      parm.revertToDefaults()

def set_properties(node):
  if (is_pdg_enabled(node)):
    return
    
  preset_index = get_preset_index(node)
  
  json_data = get_cached_json_data(node)
  asset_defs = json_data['asset_definitions']

  if (asset_defs is None or preset_index > len(asset_defs)):
    return
  
  # special 'new preset' index
  if (preset_index == len(asset_defs)):
    clear_presets(node)
    return
  

  current_asset = asset_defs[preset_index]
  
  for parm_id, value in current_asset.items():
    if (not parm_id.startswith('bg_')):
      continue
    
    current_parameter = node.parm(parm_id)
    
    if (current_parameter):
      node.parm(parm_id).set(value)
    else:
      log(node, 'Warning: found unknown parameter name in json file (' + parm_id + ')')
  
def preset_menu_callback(node):
  if (is_pdg_enabled(node)):
    return

  menu_labels = node.parm('preset_menu').menuLabels()
  
  if (not menu_labels):
    log(node, 'warning: expected there to be at least one menu label')
    return
    
  preset_index = node.evalParm('preset_menu')
  current_label = menu_labels[preset_index]

  if (preset_index == len(menu_labels) - 1):
    node.parm('preset_name').set('')
  else:
    node.parm('preset_name').set(current_label)
    

  set_properties(node)
  update_changed_params(node)
 
def force_reload(node, menu_index=-1):
  if (is_pdg_enabled(node)):
    return

  node.destroyCachedUserData('json_data')
  json_data = get_cached_json_data(node)
  
  # this will force the presets menu to rebuild itself
  force_cook(node)
  
  asset_defs = json_data['asset_definitions']
  
  if (menu_index < 0):
    node.parm('preset_menu').set(len(asset_defs))
  else:
    node.parm('preset_menu').set(menu_index)
  preset_menu_callback(node)

def update_preset(node, is_delete=False):
  if (is_pdg_enabled(node)):
    return

  preset_index = get_preset_index(node)
  
  json_data = get_cached_json_data(node)
  asset_defs = json_data['asset_definitions']

  if (asset_defs is None or preset_index > len(asset_defs)):
    return
  if (is_delete and preset_index == len(asset_defs)):
    log(node, 'error: invalid deletion index.')
    return
  
  preset_data = {}
  
  if (not is_delete):
    for parm in node.parms():
      if (not parm.name().startswith('bg_')):
        continue
      preset_data[parm.name()] = parm.eval()
    
    preset_name = node.evalParm('preset_name')
    preset_data['name'] = preset_name    
  
  if (is_delete):
    asset_defs.pop(preset_index)
    preset_index = len(asset_defs)
  elif (preset_index == len(asset_defs) or node.parm('preset_menu').menuLabels()[preset_index] != preset_name):
    # create new preset
    preset_index = len(asset_defs)
    asset_defs.append(preset_data)
  else:
    # edit existing preset
    asset_defs[preset_index] = preset_data
    
  json_data['asset_definitions'] = asset_defs
  
  # write json data
  json_file_path = node.evalParm('json_file_path')
  if (os.path.isfile(json_file_path)):
    log(node, 'writing json data to: ' + json_file_path)
  else:
    log(node, 'unable to write json data. path is invalid: ' + json_file_path)
    return
  
  json_serialized = json.dumps(json_data, indent=4)
  
  with open(json_file_path, 'w') as json_file:
    json_file.write(json_serialized)

  force_reload(node, preset_index)
```



#### Box Generator Preset Menu

```python
node = kwargs['node']

menu_options = ()

if (node.type().name() == 'hdaprocessor' or node.evalParm('pdg_enabled')):
  return menu_options
  
json_data = node.hm().get_cached_json_data(node)
asset_defs = json_data["asset_definitions"]

for index, asset in enumerate(asset_defs):
  # note: index is actually converted to a string, but it doesn't really matter,
  # because we can just grab the index of each menu item directly and ignore the "token"
  menu_options += (index, asset["name"])
    
menu_options += (len(menu_options), "New")
        
return menu_options
```



#### pdg_load_asset_defs Node

```python
def main():
  node = hou.pwd()
  generator_node = node.parent()

  if (not generator_node.hm().is_pdg_enabled(generator_node)):
    return
  
  json_data = generator_node.hm().load_json_data(generator_node)
  asset_defs = json_data['asset_definitions']
  asset_index = generator_node.parm('pdg_preset_index').eval()
  
  if (asset_index < 0 or asset_index >= len(asset_defs)):
    return
    
  asset_def = asset_defs[asset_index]
  
  geo = node.geometry()
  geo.addAttrib(hou.attribType.Global, 'scale', asset_def['bg_scale'])
  geo.addAttrib(hou.attribType.Global, 'Cd', (asset_def['bg_colorr'], asset_def['bg_colorg'], asset_def['bg_colorb']))

# ------------------------------------------------------------------
main()
```



#### Python Processor

```python
import hou
import json

node = hou.pwd()
json_data = {}

json_file_path = node.evalParm('json_file_path')
  
with open(json_file_path, 'r') as json_file:
  json_data = json.load(json_file)
  node.parm('asset_count').set(len(json_data['asset_definitions']))

item_holder.addWorkItem()
```

