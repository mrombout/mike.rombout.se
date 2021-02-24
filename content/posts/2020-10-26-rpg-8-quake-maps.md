---
title: "RPG #8 - Quake Maps"
date: 2020-10-26T20:24:00+02:00
slug: rpg-8-quake-maps
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

Every time I wanted to start on creating some actual content for the RPG I noticed it I got stuck really quickly.
For a tiled system to be effective, you need to have tiles available, I didn't.
I found myself constantly going back and forth between Blender and Godot to create and tweak new assets.
To be at the level of productivity like say when creating Neverwinter Nights maps I needed a lot work on assets, and the Godot editor as well (`GridMaps` can be quite finicky to work with).

In post [RPG #2 - Building a World]({{< relref path="2020-10-12-rpg-2-building-a-world.md">}}) I had already given up on CSG in Godot, but then I found the excellent [Qodot](https://github.com/Shfty/qodot-plugin) plugin which adds support for Quake `.map` files! And it even has built-in support for TrenchBroom by generating all the required files to get started.

I have used Valve's Hammer Editor before to create custom maps for Counter-Strike: Source which is similar to TrenchBroom.
It takes some getting used to, but once you get get a hang of it you start carving out your maps bit by bit until it has the detail you need.
And since TrenchBroom aims to be a more modern and easy to use equivalent I expect the learning curve to be much lower using that tool.

![Quake maps are imported using Qodot to create exact replicatas using meshes and collision shapes](/img/qodot-godot.gif)

So far my level imported perfectly (color variation and camera angle is my doing when taking the screenshot).
And it even comes with a framework that allows creating custom point and brush entities to use in TrenchBroom.

So far I've created a custom the following custom entities:

* Point
  * `player_start` to mark the players starting position.
  * `sound` to create an `AudioStreamPlayer` or `AudioStreamPlayer3D`.
  * `sprite` to quickly create a billboard `Sprite3D` based on the given texture.
  * `particles` to add a `Particles` node based the given `.tscn`.
* Brush
  * `hurt` to mark a zone that damages the player when they are inside it.
  * `sound_bgm` to change BGM when the player enters this area.
  * `scripted` to add a script to a brush entity.

So far I quite like this workflow as I can concentrate on carving out the level in TrenchBroom and occasionally import it into Godot to playtest it.
Any simple assets (like stalactite and stalagmites) I can just create inside TrenchBroom and they'll look good enough for the type of game I'm creating.
And for more complex assets I can temporarily block them out in TrenchBroom and create proper assets of them later.
