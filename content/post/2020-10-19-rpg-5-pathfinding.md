---
title: "RPG #5 - Pathfinding"
date: 2020-10-19T18:50:00+02:00
slug: rpg-5-pathfinding
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

In western RPGs enemies usually have a visual representation in the game world (unlike many jRPGs).
This means that these visual representations need to move around on their own, otherwise they wouldn't be believable.
Especially if I want to implement in-world combat, having enemies know how to navigate the world and move towards their targets would be helpful.

Godot offers AI pathfinding through the [`Navigation`](https://docs.godotengine.org/en/stable/classes/class_navigation.html) node.
Unfortunately it appears to be designed for todays most common scenario where nodes will navigate freely through the available 3d space marked using NavMeshes.

So far I've been building the RPG to have grid-based movement and hence, such navigation doesn't make too much sense.
Luckily Godot also contains a `AStar` a generic implementation of the [A* search algorithm](https://en.wikipedia.org/wiki/A*_search_algorithm).

## Implementation

The `AStar` class is a ready to use implementation of the A* algorithm.
All you need to do is fill it with points and connect these points up to create a [graph](https://en.wikipedia.org/wiki/Graph_(discrete_mathematics)).

I implement the first version of my `GridNavigation` node following the design of the `Navigation` node.
Where a `GridMap` takes place of the `NavMesh` so to speak.

The `GridNavigation` component will then process the `GridMap` to create a graph using the `AStar` class using code similar to the following (pseudo-code):

```
func _ready():
  create_all_points()
  connect_all_points()

func create_all_points():
  for x in range(-50, 50):
    for y in range(-50, 50):
      var cell_item = gridmap.get_cell_item(x, 0, y)
      if cell_item == GridMap.INVALID_CELL_ITEM:
        add_point_to_astar()

func connect_all_points():
  for point_id in astar.get_points():
    var east_point_id = astar.get_closest_point(Vector3(point_position.x + 1, 0, point_position.z))
    if east_point_id != -1 and point_id != east_point_id:
      astar.connect_points(point_id, east_point_id, true)

    var south_point_id = astar.get_closest_point(Vector3(point_position.x, 0, point_position.z + 1))
    if south_point_id != -1 and point_id != south_point_id:
      astar.connect_points(point_id, south_point_id, true)
```

This will take the 0th layer of the GridMap and connect up all free spaces through points in the `AStar` class instance.
Once all points are added and connected the `astar` can be queried for the fastest route towards a certain point.

Further improvements could be made such as adding weights based on tile to make the AI prefer bridges (lower weight) over water (higher weight) for example.
