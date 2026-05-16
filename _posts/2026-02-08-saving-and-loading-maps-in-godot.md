---
layout: post
title: Saving and Loading Maps in Godot - the Smart Way!
description: How to put GDScript's reflection system to good use.
tags: [godot, game dev, tutorial]
categories: [Godot]
date: 2026-02-09 18:08 +0100
media_subpath: /assets/img/posts/saving-loading-maps-godot
image: thumb.png
---

I am currently working on a top-down survival game in Godot consisting of many maps. Within each map are several nodes with properties that need to be saved, like the boolean `looted` property for a chest. The player is free to switch between maps. Without a proper system in place to save and load node properties, the player could reload a map over and over again to keep harvesting the same resources, defeating the point of the game. I needed a system that was scalable, easy to use, and future-proof. In this tutorial, you will learn how to implement such a system, dubbed the “**Scene Storage**”, yourself.

> Scene Storage has only been tested with Godot 4.6. There is no guarantee that it will work in older versions.
{: .prompt-warning}

## How Scene Storage Works and Who Should Use It

Before we get into the implementation details, let's briefly go over how the Scene Storage works from an architectural perspective. Then, I will list the criteria a game should have that warrant implementing such a system. After all, there isn't a one-size-fits-all approach when it comes to programming. Your game may be able to use a much simpler system. More on that later.

### Architectural Overview

Every time a new map load is requested, the Scene Storage performs the following steps:

- Retrieve all nodes that need properties to be persisted
- For each node:
  - Exclude irrelevant properties
  - Save the name of each property and its value in a dictionary
  - Save the dictionary containing all properties in a global dictionary
- Load the new map from a scene file
- Remove all nodes with persistent properties
- Add each node back in, based on its save data:
  - Instantiate the node
  - Apply the saved properties
  - Add it back into the scene at the correct position

> Reading this, you might be wondering: why not just apply the properties to the already existing nodes? Several of my nodes have code in `_ready()` that changes their appearance based on property values. For instance, a chest node with `var looted = true` will display an open chest. If I apply properties after loading a scene, they will be applied **AFTER** `_ready()` has been called. So the chest would appear closed, despite already being opened. Therefore, it's easiest to delete the nodes and add them back in, applying the properties before they enter the scene tree.
{: .prompt-info}

### Should You Use It?

Next, let's go over who should use this system. If your game:

- has scenes that contain many nodes with properties that need to be saved
- has non-linear progression, i.e., things can happen in various orders
- is split up into several maps that are completely removed when the player exits them

then you might benefit from this system. Should your game have more linear progression and fewer things to save, then a simple list of properties in a dedicated `Resource` could be good enough. If your game has fully procedural maps, you might be better off using `PackedScene`s. I will not explain these approaches in this tutorial, so feel free to research them yourself. If you're unsure right now, don't worry. Read on and see whether this system is appropriate or overkill for your project.

## Saving Node Data

Firstly, we need to get all nodes that need their properties saved. I achieve that with Godot's group feature. All nodes relevant to Scene Storage are added to a global group, “Persistent”. Then I can just call `get_tree().get_nodes_in_group("Persistent")` to access them.

Next up, I loop through all nodes and save their properties, along with some other information. This data will be used to recreate the node when the map is loaded again.

### Properties for Re-Instantiation

This is the code for storing the node's name, parent path, and scene file path or class name:

```gdscript
node_data: Dictionary = {}
node_data.set("name", node.name)
node_data.set("parent_path", node.get_parent().get_path())

if not node.scene_file_path.is_empty():
  node_data.set("scene_file_path", node.scene_file_path)
elif node.get_script():
  var script = node.get_script() as Script
  if script.get_global_name().is_empty():
    node_data.set("native_class", node.get_class())
  else:
    node_data.set("custom_class", script.get_global_name())
```
{: file="scene_storage.gd"}

What's going on here? Consider how you can create custom nodes. Either you create a scene file and instantiate it, or you define a node purely by script. This code handles both cases. For script-based nodes, there are different ways to retrieve a built-in class (like `Area2D`) and a custom one, defined with `class_name` at the top of the script. Consider the two nodes in the screenshot below as an example. “Brittlebush” is a scene instance (as denoted by the clapboard) icon. “BuildingGrid”, on the other hand, is only defined by a script with a custom class (hence the grayed-out script icon).

![Godot Scene Outline with two nodes. “Brittlebush” is an instance of a scene, whereas “BuildingGrid” is not.](node-types.png)
_“Brittlebush” is an instance of a scene, whereas “BuildingGrid” is not._

### Other Node Properties

Next up, this is how I store most other node properties:

```gdscript
var property_list = node.get_property_list()
for property_info in property_list:
  # Optional: Only save properties that are marked for storage (with @export or @export_storage annotation).
  if not property_info["usage"] & PROPERTY_USAGE_STORAGE: continue

  var name = property_info["name"]
  var property = node.get(name)
  node_data.set(name, property)
```
{: file="scene_storage.gd"}

With that, `node_data` contains all the data we need. We can then save it in another dictionary, `save_data`, alongside the data for all other nodes:

```gdscript
var save_data: Dictionary = {}
# Node saving code...
save_data.set(node.get_path(), node_data)
# Repeat for each node
```
{: file="scene_storage.gd"}

For my system, I use the node's `NodePath` instead of its name. Two nodes can have the same name but not the same path, so this approach is more robust. It comes with some gotchas that I will discuss during the next section. Let's move on to loading a new map!

## Loading A New Scene

After loading a new scene with `get_tree().change_scene_to_file()` or a similar method, we need to delete all persistent nodes first:

```gdscript
get_tree().change_scene_to_file(path)
await get_tree().scene_changed # Wait for scene to be ready
var nodes = get_tree().get_nodes_in_group("Persistent")
for node in nodes:
  node.free()
```
{: file="scene_storage.gd"}

Note that I do not use `queue_free()`. Most resources, including the official Godot documentation, will tell you to use this method over `free()`. It's generally safer because it does not immediately delete the node. It merely marks the node for deletion at the end of the frame. It's still in the scene tree and valid to access until it is deleted later on. This, however, is problematic for our system.

Recall that we saved the node's name in the previous section. The newly created nodes will share the names of the nodes that were deleted. This is to keep any `NodePath`s pointing to the deleted nodes valid; they are simply replaced by new ones. If we were to use `queue_free()` instead, the following would happen:

- Old node is queued for deletion with `queue_free()` - it's still in the scene tree.
- The new node is instantiated and named after the old node.
- The old node still exists, so Godot adds a number to the new node's name to resolve the naming conflict.
- The old node is finally deleted, and any `NodePath`s pointing to it are now invalid, since the new node has a different name.

To avoid this, we simply immediately delete all old nodes. It is safe to do so, as long as we don't try to access any of them later.

### Instantiating New Nodes

Next, we need to recreate all the nodes again, only using the data that we've saved for each node. Iterate through each key of `save_data` and instantiate each new node:

```gdscript
var new_node: Node
if node_data.has("scene_file_path"):
  new_node = load(node_data["scene_file_path"]).instantiate()
elif node_data.has("native_class"):
  var cls_name = node_data["native_class"]
  if not ClassDB.class_exists(cls_name): continue
  new_node = ClassDB.instantiate(cls_name)
elif node_data.has("custom_class"):
  var cls_name = node_data["custom_class"]
  var class_list = ProjectSettings.get_global_class_list()
  var list = class_list.filter(func(entry): return entry["class"] == cls_name)
  if list.is_empty(): continue
  new_node = load(list[0]["path"]).new()
```
{: file="scene_storage.gd"}

If you thought the code to save the scene file path or node class was messy, this code is far worse. It is, however, necessary to correctly instantiate a node of any kind. If the node data has a scene file path, it's easy: just load the scene and instantiate it like you normally would. If the node doesn't have a scene file associated, things are more tricky.<br>
For internal classes like `StaticBody2D` and `Area3D`, `ClassDB.instantiate(name)` can be used to easily instantiate the class. For custom classes, we first need to retrieve the list of custom classes from the `ProjectSettings` singleton. Each entry is a `Dictionary` containing the class name (key: `"class"`) and the path to the file defining the class (key: `"path"`). Find the correct class and then use its path to create a new instance.

> This code does not work for classes with custom `_init()` functions that require properties, because those need to be passed into `new()`. Fixing this is left as an exercise for the reader, meaning I'm too lazy to do it myself.
{: .prompt-warning}

### Applying Node Properties

With the node now (hopefully) instantiated, we can apply its properties, including its name:

```gdscript
node.name = node_data["name"]
for property_name: String in node_data.keys():
  var property_data = node_data.get(property_name)
  new_node.set(property_name, property_data)

var parent_path = node_data["parent_path"]
if Game.has_node(parent_path):
  Game.get_node(parent_path).add_child(new_node)
else:
  Game.get_tree().current_scene.add_child(new_node)
```
{: file="scene_storage.gd"}

> This system doesn't handle persistent nodes that are children of other persistent nodes. It assumes the parent is already there. This is once again left as an exercise for the reader. My scene trees are relatively flat, so this isn't an issue for me (yet).
{: .prompt-warning}

With that, the node has been successfully set up and added into the scene tree. Once this is done for all nodes, the scene should function as expected, and the nodes should have the correct property values.

## Warnings and Issues

As you may have noticed, I have added a couple of disclaimers in the previous sections. The system, while useful and functional, is far from perfect:

- Persistent nodes cannot have persistent children in its current iteration
- Scripts with custom `_init()` functions do not work
- Nodes that reference persistent nodes without using `NodePath`s will have invalid references after load. This system requires the use of `NodePath`s.
- I ran into a weird issue where a saved property of type `Dictionary[Vector2, Variant]` would have its values nulled. I had to change the type to `Dictionary[Vector2, NodePath]` and fake null values.
- Probably more that I've forgotten

Some of these issues can be fixed (like the parenting issue); others require workarounds, unfortunately. Still, if you consider these trade-offs worth it, feel free to make use of my code.

## Addendum: Saving Node State to Disk

Before I wrap up this post, I wanted to quickly go over saving the data to disk. My game's odd design doesn't require this, but there is a chance yours will. The data can be easily saved by putting it in a custom `Resource`. Then, you can use `ResourceSaver.save()` to easily save it to disk so the data persists between play sessions:

```gdscript
class_name SaveData
extends Resource

var data: Dictionary[String, Dictionary]
```
{: file="save_data.gd"}

```gdscript
var save_data = SaveData.new()
save_data.data = add_your_dict_here
var save_path = "user://save_data"

save_data.take_over(save_path) # Might be necessary to overwrite an already existing file.
ResourceSaver.save(save_data)
```
{: file="save_data_saver.gd"}

Loading it back into the game can be done with a simple `load()` or, for more complex cases, `ResourceLoader.load()`:

```gdscript
var save_path = "user://save_data"
var save_data = load(save_path)

save_data.data # Do something with this.
```
{: file="save_data_loader.gd"}

## Wrapping Up

And this is how you can implement a general-purpose map-saving and loading system in Godot. It can be adapted and improved upon further, but I hope someone can benefit from the groundwork I've put in! It would make me very happy. If anything is unclear or isn't working, please let me know in a comment, and I will answer as quickly as possible.

Until then, take care and keep on creating.

\- Lennart
