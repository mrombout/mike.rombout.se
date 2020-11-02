---
title: "RPG #9 - Free Movement"
date: 2020-11-02T01:13:00+02:00
slug: rpg-9-free-movement
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

Since I'm now using TrenchBroom and Qodot is create and import my levels, having grid-based movement doesn't feel right anymore.
Besides, very few old-school first-person RPG still used grid-based movement once they went 3D as far as I know.
So let's rewrite the movement to be more like Might and Magic VI!

## Implementation

As I haven't worked with real Godot's physics before I started with the [FPS tutorial](https://docs.godotengine.org/en/stable/tutorials/3d/fps_tutorial/index.html) available in the official docs. And it provided a good basis but had features I didn't want like mouse-look and interpolated velocity (it wasn't clunky enough for a old-school RPG!), so I removed all of that code.

### Controllers

One major alteration I did was that instead of writing all the code in the `KinematicBody` player script I added the script to a child `Node` which takes a target player to operate on.
This way I managed to split out the mouse/keyboard and controller controls into separate scripts.
Instead of modifying the body directly, they call methods to indicate their intention such as `move(input_movement: Vector3)`, `jump()`, `start_sprinting()`, etc.

![Scene tree of player scene with three separate controller nodes](/img/player_controller_tree.png)

With this I was basically trying to replicate the `Pawn` and `Controller` relation you have in Unreal Engine.
Since my controllers operate on a target I could even move them outside of the `Player` scene.

Then I started on a completely separate controller to try and mimic the controls of Might and Magic VI, which turned out to be quite easy since I had already finished most of the heavy lifting by simply following the tutorial.

![Demonstration of player tank controls](/img/moving_around.gif)
