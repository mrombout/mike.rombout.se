---
title: "RPG #26 - Objects dock"
date: 2022-05-20T18:07:00+00:00
slug: rpg-26-objects-dock
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

In the previous blog post I implemented a custom importer for my `.mod` files.
Unfortunately, that didn't solve my initial problem of having to go out of Godot to search the contribution database for IDs.

In order to solve that I want to implement a custom `Objects` dock in Godot that will load all the contribution from a `.mod` file a shows it in a nice way.
It should also allow me to filter on ID so that I can quickly search for IDs in the database.

## Custom dock

Adding a custom dock in Godot is very easy.
All you need to do is a create a `Control` (could be a Scene) and add it to the UI by using the `add_control_to_dock(..., ...)` function.

### Create control

I started by creating the `Control` as a Scene by adding a couple of standard built-in control nodes.

* `Objects: VBoxContainer`
  * `MenuContainer: HBoxContainer` - will contain a menu toolbar for custom actions to perform on selected nodes.
    * `FileButton: MenuButton` - at the moment shows an `Open...` and `Save all` action.
  * `FilterLineEdit: LineEdit` - allows filtering the loaded module contributions.
  * `ResourceTree: Tree` - used to show all contribution in a module.
  * `PopupMenu: PopupMenu` contains a contextual menu when clicking on a contribution item in the `ResourceTree`.

![DoD Objects Dock scene tree](/img/26-objects-dock-scene-tree.png)

In the script I added a `load_module(module: Module) -> void` function that reads a `Module` resource and creates tree items for each contribution.
It's pretty straight forward.

### Add control

After that I added it as a control in my `plugin.gd`'s `_enter_tree` function.

```gdscript
tool
extends EditorPlugin

const ObjectsDock = preload('res://addons/dod_editor/ObjectsDock.tscn')

var objects_dock

func _enter_tree():
  var module = load('res://modules/main/module.mod')

  objects_dock = ObjectsDock.instance()
  objects_dock.plugin = self
  objects_dock.editor_interface = get_editor_interface()
  add_control_to_dock(DOCK_SLOT_LEFT_BR, objects_dock)
  objects_dock.load_module(module)
```

After reloading Godot the dock is visible and showing all the contributions in the `module.mod` file.

![DoD Objects Dock](/img/26-dod-objects-dock.png)

### Search

In order to implement the search I want to only show contributions that match the given filter string.
Unfortunately I couldn't find a way to hide a `TreeItem`, so instead I am rebuilding the entire tree when a filter is applied.

```gdscript
func _update_tree_items():
  # ... omitted for clarify

  # create root for map contributions
  var map_root_item = master_tree.create_item(module_root_item)
  map_root_item.set_text(0, "Maps")
  map_root_item.set_icon(0, MapIcon)
  map_root_item.set_icon_modulate(0, Color.greenyellow)
  map_root_item.collapsed = true
  
  # load each map contribution
  for map in module.maps:
    # only create a tree item if it matches the filter, if there is one
    if filter and not filter in map.id:
      continue
    
    var map_sub_item = master_tree.create_item(map_root_item)
    map_sub_item.set_text(0, map.id)
    map_sub_item.set_meta('type', 'map')
    map_sub_item.set_meta('module', module.get_name())
    map_sub_item.set_meta('id', map.id)
    map_sub_item.set_meta('resource', map)
    map.connect('changed', self, '_on_resource_changed', [map_sub_item, map])

  # ... omitted for clarify
```

Now, whenever I am looking for an ID I can either look through the category.
Or better yet, if I remember part of the ID I can do a search and it will show all contributions that match.
So if I would be looking for something like `bridge` the result would be as follows:

![DoD Objects Dock search](/img/26-dod-objects-dock-search.png)

## Conclusion

Being able to quickly search for IDs in the objects dock is already helping me out in some cases.
It also provides a good place for other conveniencies.
For example, double-clicking on a `Map` resource will open up the map in the 3D editor.

I think the Objects dock will play a large part in working on content.
From here I want to be able to manage resources and interact with them in a meaningful way like placing items and entities in the 3D editor, opening up a map from a marker, etc.
