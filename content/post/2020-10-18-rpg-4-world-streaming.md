---
title: "RPG #4 - World Streaming"
date: 2020-10-18T12:07:00+02:00
slug: rpg-4-need-bigger-world
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

Huge worlds to explore has been a staple for RPGs as early as 1981 in Ultima.

Unfortunately it's not always possible to load such massive worlds into memory, even on todays computers where game worlds become more and more complex.

Even though I haven't ran into performance issues yet I decided to already build a world streamer so that I can potentially build infinite worlds.

## Implementation

For my first implementation I took the simplest approach I could come up with and implemented the following algorithm (pseudo code):

```gdscript
func _on_player_moved():
  var chunk_x = int(floor(world_x / 100))
  var chunk_y = int(floor(world_y / 100))

  load_chunk(center)
  for direction in directions:
    load_chunk(direction)
  
  remove_old_chunks()

func load_chunk(direction):
  if !has_node(direction):
    var chunk = load_chunk().instance()
    add_child(chunk)
```

The animation below demonstrate how it loads 9 chunks of the world wherever the player is in the world. As if now the world is 100 by 100 tiles, but I might be able to load larger tiles after I have done some testing.

![Animation how new chunks are loaded as the player moves from chunk to chunk](/img/map_streaming.gif)

In order to get rid of the flickering I would have to make some improvements like loading tiles in the background, caching tiles and preloading tiles.
