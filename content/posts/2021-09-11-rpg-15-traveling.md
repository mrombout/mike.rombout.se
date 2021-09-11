---
title: "RPG #15 - Traveling"
date: 2021-09-11T20:11:00+02:00
slug: rpg-15-traveling
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

In [RPG #13 - Big update of small updates]({{< relref path="2021-04-29-rpg-13-big-update.md" >}}) you may have seen a quick glimpse of the "world travel screen".
I have to admit this is just a compromise because building a full open-world RPG might be a bit too much.
Also, the implementation is pretty simple and boring, but it's still a pretty nifty feature to show off by itself I suppose.

And it's heavily inspired by Fallout's way of handling traveling.
As you travel along the map in Fallout your character moves and encounters may happen after which you are teleported to a gameplay map.
This gives the scale of a large open-world game, without having to actually design a massive open-world map.

![Screenshot of Fallout's world map](/img/fallout_world_map.jpg)

In my implementation it works in quite the same way.
The player clicks on a spot on the map or a point-of-interest after which the character will slowly start moving towards that point.
At random intervals a number of random events may happen (which one and how often depends on a number of factors).

![Screenshot of Draeywins's world map](/img/world_map_event.png)

While the implementation is nothing interesting, it does make use of a couple of nifty features that come with Godot out of the box like `Navigation2D` which is used to find the path.
Unfortunately, I could not use `Line2D` as it appears to draw lines in a resolution-independent way, which resulting into smooth lines, so I had to draw the line myself by iterating over the points and using `draw_line()`.