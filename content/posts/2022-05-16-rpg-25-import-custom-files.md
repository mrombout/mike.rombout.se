---
title: "RPG #25 - Import custom files"
date: 2022-05-15T18:07:00+00:00
slug: rpg-25-import-custom-files
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

For the last couple of months I had been working on creating content for my RPG.
As I was working on it, I noticed that I was often distracted by cumbersome boilerplate and having to look up IDs in the database.
So I my first Godot editor plugin I wanted to make Godot better aware of the data in my game, and help me find IDs (and possibly validate them) as quickly as possible.

## Resources

The moment I started investigating, I noticed (in hindsight) I made a mistake in my design.
A lot of data in my game is static, so for that I decided to create a JSON file that I read upon game start to build up an in-memory database that can be queried from script.
It works fine, but by using raw JSON this way I'm actually not using Godot to its full potential.
It should have been a [`Resource`](https://docs.godotengine.org/en/3.4/tutorials/scripting/resources.html#resources).

In Godot a `Resource` is their native format for pretty much anything that is loaded or saved to disk.
The engine comes with a lot of resources built-in, but it's also possible to [create your own](https://docs.godotengine.org/en/3.4/tutorials/scripting/resources.html#creating-your-own-resources).
And doing so has several benefits, with the most important benefits for me being:

* Godot Engine's Inspector renders and edits Resource files out-of-the-box. As such, users often do not need to implement custom logic to visualize or edit their data. To do so, double-click the resource file in the FileSystem dock or click the folder icon in the Inspector and open the file in the dialog.

But the following could prove to be very nice to have as well:

* Resource auto-serialization and deserialization is a built-in Godot Engine feature. Users do not need to implement custom logic to import/export a resource file's data.
* Resources can even serialize sub-Resources recursively, meaning users can design even more sophisticated data structures.
* Users can save Resources as version-control-friendly text files (`*.tres`). Upon exporting a game, Godot serializes resource files as binary files (`*.res`) for increased speed and compression.

So using resources would allow be to edit them using the Inspector dock like Godots built-in resources.
It would also allow be to easily serialize, and de-serialize them in either text-based files, or binary files.
Unfortunately, it also means I have to use Godot's custom `*.tres` or `*.res` formats, which was the reason I opted for plain old JSON early on in development.

After digging around though I found a way where I could still use my JSON format, while also reaping some of the benefits of custom resources.
All I needed to do was to write a custom importer!

## Importer

In Godot an `Importer` is responsible for turning an external resources (e.g. a `.png` texture, or `.wav` audio file) into a Godot resource.
If you want to add support for a new `Texture` format, or a new audio format for an `AudioStream` you could do so with an [`EditorImportPlugin`](https://docs.godotengine.org/en/stable/classes/class_editorimportplugin.html#class-editorimportplugin).
But this is also true for custom resources.

So what if I pretend my `module.json` files are special, and are to be imported as my custom `Module` resource?
That way I could just import my `module.json` to turn it into a resource so that the Godot editor understands the data inside it, and I can just serialize that into a binary `*.res` file while compiling the game to get a performance boost.

### Custom resource

I started by creating custom resources by creating scripts that extend `Resource`, matching all properties that they had in JSON.
This is very easy to do, for example the `Module` root resource looks like so:

```gdscript
tool
extends Resource

export(String) var name
export(Array, Resource) var maps 
export(Array, Resource) var markers
export(Array, Resource) var quests
export(Array, Resource) var events
export(Array, Resource) var containers
export(Array, Resource) var entities
export(Array, Resource) var items
```

> NOTE: Unfortunately, Godot doesn't seem to allow using custom resources names for `export` as of yet. But because I am just playing to use these resources for the importer it's not too problematic for me.

### Module importer

```gdscript
extends EditorImportPlugin

enum Presets { DEFAULT }

const ModuleResource = preload('res://addons/dod_editor/resource/ModuleResource.gd')
const MapResource = preload('res://addons/dod_editor/resource/MapResource.gd')

func get_importer_name():
  return "dod.module"

func get_visible_name():
  return "DoD Module"

# I renamed the file to `mod` so that it doesn't try to import all JSON as
# modules. I'm sure there's a better way to avoid this.
func get_recognized_extensions():
  return ["mod"]

func get_save_extension():
  return "tres"

func get_resource_type():
  return "Resource"

func get_preset_count():
  return Presets.size()

func get_preset_name(preset):
  match preset:
    Presets.DEFAULT:
      return "Default"
    _:
      return "Unknown"

func get_option_visibility(option, options):
  return true

func get_import_options(preset):
  match preset:
    Presets.DEFAULT:
      return []
    _:
      return []

func import(source_file, save_path, options, platform_variants, gen_files):
  # read file as normal
  var source_module_file = File.new()
  var err = source_module_file.open(source_file, File.READ)
  if err != OK:
    return err

  # parse file as JSON
  var source_module_json_string = source_module_file.get_as_text()
  var source_module_parse_result = JSON.parse(source_module_json_string)
  if source_module_parse_result.error != OK:
    return err
  
  # create ModuleResource and fill with extracted data
  var source_module_dict = source_module_parse_result.result
  
  var module_resource = ModuleResource.new()
  module_resource.name = source_module_dict.name
  
  # for each map create a MapResource and fill with extracted data
  if source_module_dict.has('maps'):
    for map in source_module_dict.maps:
      var module_map_resource = MapResource.new()
      module_map_resource.type = map.type
      module_map_resource.id = map.id
      module_map_resource.name = map.name
      module_map_resource.res = map.res
      module_map_resource.resource_name = map.id
      
      module_resource.maps.append(module_map_resource)
  
  # ... omitted code for many more resources
  
  # save the resource **as** a resource
  return ResourceSaver.save("%s.%s" % [save_path, get_save_extension()], module_resource)
```

By adding this code and registering it as an imported in my Godot plugin all `.mod` files will automatically be imported when I open the editor, a new importer will also appear whenever I select my `.mod` files.

![Dod Module Importer](/img/25-dod-module-importer.png)

Now whenever I select a `.mod` file that has been imported it will also show up in the `Inspector` dock.

![Dod Module Importer](/img/25-dod-module-inspector.png)

Without any additional effort I can now see all the data in `.mod` file through Godot's `Inspector` dock.
It's pretty nice, but by itself it is not yet very useful.
It does however provide a good platform for all other editor plugin ideas that I have.

### Modifying `.mod` files

Now that I've written an importer Godot "understands" my custom format better.
My importer reads the input JSON, and writes in Godot's own resource format (which in case of `*.res` is considered more efficient than text or JSON).

But because it's based on a `.mod` (which is JSON) I can't modify the resources through the inspector.
In order to do that I've added some additional code, and a custom "Save" button to serialize changes made in the `Inspector` dock back to the `.json`.

```gdscript
tool
extends Resource

# ... exports omitted

func save():
  var dict := to_dict()
  print("saving to %s" % [resource_path])

  var file := File.new()
  file.open(resource_path, File.WRITE)
  file.store_string(JSON.print(dict, "\t"))
  file.close()
```

## Conclusion

All `.mod` files are now "understood" by Godot.
I can import `.mod` files and inspect and modify the data through the `Inspector` dock.

I still have to look for IDs, but at least I don't have to leave Godot in order to do so.
In the future however, I want to write a custom dock plugin that shows all the items in the database which allows searching using basic string filters and regular expressions.
