---
title: "RPG #2 - Building a World"
date: 2020-10-12
slug: rpg-2-building-a-world
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

One of the most important things in RPGs for me is exploration.
So let's start with implementing a representation of the world that we can move around in.

One major benefit of engines like Godot is that we have an editor available to use right away.
So technically, we could just create a [`MeshInstance`](https://docs.godotengine.org/en/stable/classes/class_meshinstance.html) with a [`BoxShape`](https://docs.godotengine.org/en/stable/classes/class_boxshape.html) and call it a day.

And while that probably works really well for people that are good with Blender I prefer to have something more restricting so that I can focus on actually building a level rather than fiddling with details for way too long.

I considered the following options:

* [Blender with Level Buddy](https://matt-lucas.itch.io/level-buddy)
* [CSGPrimitives](https://docs.godotengine.org/en/stable/tutorials/3d/csg_tools.html)
* [GridMaps](https://docs.godotengine.org/en/stable/tutorials/3d/using_gridmaps.html)

## Blender with Level Buddy

Blender can do everything, and for a lot of people level editing is one those things too.
Unfortunately I found it's really not for me after trying to give it a short for level editing.

Although I am a terrible modeler I'm quite comfortable using Blender for non-organic modeling.
But the vast amount of options Blender has just distracts me, spending too much time on unimportant details.

Additionally, I would probably need to write a Blender addon in order to add game information like enemy placement, scripted elements and things like that.

So I'm not going with this option for now.

## CSGPrimitives

Constructive solid geometry (CSG) is a technique of modeling that was quite popular in older games like Quake.
In gaming this can be used to quickly create interesting shapes using only a couple of primitives that you combine to merge, cut or mask other objects.

Unfortunately I found Godot's implementation a bit too finicky to work with since you can only scale shapes from the center rather than pulling sides out like you can in [TrenchBroom](https://kristianduske.com/trenchbroom/).

So I'm not going with this option for now.
Although I wish I could.

## GridMaps

GridMaps in Godot are basically just 3D tilemaps by placing 3D models of a fixed square grid.

One benefit of this approach is that once I've built the TileSet I can start building the map without getting distracted by any details.

Since this seemed to be the way that requires the least effort I decided to implement this at least until something better comes along.

## Implementation

While [Using gridmaps](https://docs.godotengine.org/en/stable/tutorials/3d/using_gridmaps.html) provides a good example I had a little trouble finding out how to effectively create a `MeshLibrary`.
It turned out to be quite simple:

1. Create a scene and add one `MeshInstance` for each tile you need.
    * Only `MeshInstance` nodes are considered tiles.
    * If you want collision, add the collision nodes inside the `MeshInstance`.
2. Create a **New Resource...** and select `MeshLibrary`
3. In the main editor window, click **Mesh Library > Import from Scene** and select the scene you created in step 1.
4. Your tiles will now appear under **Item** in the **Inspector**.

After that by placing tiles, rotating them around I was able to pretty quickly build up a little town.

![A simple town using GridMap](/img/building_a_world.png)

For moving around I built a quick player class that moves in steps aligned with the grid and turns 90 degree angles.
Then I interpolated the players position to create smooth movement and turns.
It feels rather clunky and needs a lot of work, but at least I was able to move around the little town that I built!

![Moving around the GridMap world](/img/moving_around.gif)
