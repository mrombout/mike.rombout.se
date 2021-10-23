---
title: "RPG #17 - Open worlds are back"
date: 2021-09-30T20:42:00+02:00
slug: rpg-17-open-worlds-are-back
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

In [RPG #4 - World Streaming]({{< relref path="2020-10-18-rpg-4-world-streaming.md" >}}) I had implemented a world streamer.
A scene that loads and unloads chunks as the player moves around a world that would be too big to load in the memory all at once.

Quickly there-after though I decided to focus on building high-quality dungeons since I figured creating an massive open-world RPG right from the start would be too much of a challenge though.
But as I've been building maps and loading them as dungeons I've started to run into loading issues for some of the bigger cities I've been creating.
So this had be looking back at the world streamer to see if I could use this for larger maps as well.

So rather than to create an entire massive world at once, I'm not using the worlds streamer for maps that are too big to load as dungeons.

## Challenges

But I've quickly found that creating world streamer maps was cumbersome.
I need to do a lot of copy and pasting to create a grid of chunk scenes.
Then I need to open each chunk in TrenchBroom and build the map, but to see the final result I need to compile each scene in Godot.
And then to see how they integrate I need to open the world streamer, load the map I'm working on and move the character around.

> It's difficult to see the big picture when working on chunks individually.

At the moment, building my own level-editor is too much work (I'd rather work on finishing the game), so I decided to build something much simpler.

## World renderer

To help me get an overview of what I'm working on I've created a `worldrender` tool that renders each chunk in a map with an orthogonal camera and stitches all of them together into one big image.
While I still need to edit each chunk individually, and build them in Godot, it's now much easier to iterate by simply re-rendering the world as an image.

![Screenshot of Draeywin's introduction island](/img/world_map_render.png)

Above you can see the produced result (reduced by 50%), in this image each 285x285px is a chunk.
As you can see, I've blocked out most of the landmass for this island, but put in the details for only a couple of chunks.

Nevertheless, designing these massive maps is now much easier after only an hours work of building this tool.

### Implementation

I've hacked this together pretty quickly (it's only a tool after all, right), but it does what I want it to do right now.
It consists of two parts, a Godot scene that renders the chunk to `.png`s and a Go utility that stitches them together into one big `.png`.

```gdscript
extends Spatial

# $Camera is an orthogonal camera
onready var camera_node: Camera = $Camera

func _ready():
  # load the map specified by INPUT
  var map_path = OS.get_environment("INPUT")
  var map_scene = load(map_path).instance()
  call_deferred("add_child", map_scene)

  # yield (because add_child is deferred)
  yield(get_tree(), "idle_frame")
  yield(get_tree(), "idle_frame")

  # take a screenshot and quit
  _take_screenshot()
  get_tree().quit(0) # quit once we're done to the utility can start the next one

func _take_screenshot():
  var camera_image := camera_node.get_viewport().get_texture().get_data()

  # crop to take only the relevent part (hard-coded for now)
  var cropped_image = Image.new()
  cropped_image.create(570, 570, false, camera_image.get_format())
  cropped_image.blit_rect(camera_image, Rect2(355, 195, 570, 570), Vector2(0, 0))
  cropped_image.save_png(OS.get_environment("OUTPUT"))
```

And then the Go utility kicks on Godot with the `INPUT` and `OUTPUT` environment variables set for each chunk in the world and then stitches them together using `image` and `image/draw`.
By using `--no-window` Godot doesn't start a window so that it can run in the background without messing with my workflow while I wait.

```go
rendererCmd := exec.Command("godot", "--upwards", "--no-window", "Renderer.tscn")
rendererCmd.Env = os.Environ()
rendererCmd.Env = append(rendererCmd.Env, fmt.Sprintf("INPUT=%s", inputPath))
rendererCmd.Env = append(rendererCmd.Env, fmt.Sprintf("OUTPUT=%s", outputPath))
```

At the moment if an `OUTPUT` image already exists, it won't render a new one.
This allow me to selective render an update chunk of the map by simply deleting that `OUTPUT` chunk.

### Improvements

Having this map is already a great help, and I might even use it in-game later to implement the world map.
There are still a couple of things that I would like to improve:

* Option to render a grid on-top of the output to make it easier to identify which chunk to modify.
* Improve the speed, at the moment it takes a long time to render a 10x10 map.

I'm also thinking of implementing other tools such as:

* Generate a world from a heightmap and texturemap.
  * Separate the heightmap into chunks.
  * For each chunk, generate a Quake map with the right brushes.
  * For each chunk, apply the right texture.
* Automatically compile (with Qodot) all map chunks in a world.

## Conclusion

Quick and dirty command-line tools like this can very effective.
There is no need to build complicated GUI-based full-blown level-editors right from the start.
