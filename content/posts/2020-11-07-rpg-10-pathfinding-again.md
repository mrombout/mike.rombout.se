---
title: "RPG #10 - Pathfinding (again)"
date: 2020-11-07T09:24:00+02:00
slug: rpg-10-pathfinding
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

In the [last blog]({{< relref path="2020-11-02-rpg-9-free-movement.md">}}) I switched from a grid-based movement system to a system that allows the player to move in any direction.
Additionally, I've stopped using a `GridMap` for build my levels and instead am now using the [Quake map format]({{< relref path="2020-10-26-rpg-8-quake-maps.md" >}}).

Unfortunately this means that [the pathfinding system using AStar I built before]({{< relref path="2020-10-19-rpg-5-pathfinding.md">}}) is no longer valid.
In contrast though, I can now use `Navigation` nodes for pathfinding without doing too much work myself.

## Implementation

Qodot generates collision meshes for us, this means that if we simply move the `QodotMap` to be a child of `NavigationMeshInstance` we can automatically generate the `NavMesh` from the level.

![Tree when using Qodot with Navigation](/img/navigation_tree.png)

From there I retrofitted the script from my previous implementation so that monster would follow an array of points to their target.
For now updating the target every frame doesn't appear to be a problem with small amount of monsters.
