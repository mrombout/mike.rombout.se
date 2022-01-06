---
title: "RPG #23 - Reflection on Godot"
date: 2022-01-76T08:53:28+01:00
slug: rpg-23-reflection-on-godot
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

At the time of writing this blog it has been over a year since I've started writing my personal development diary in public (AKA devblog) about the RPG I am developing in [Godot Engine](https://godotengine.org/).

I've survived a major release of the engine going from `2.1.x` to eventually `3.4.2`, and `4.0.0` is coming.
So I figured it's about time I reflect on my experience with Godot so far, and see where the future is heading.

## Why Godot though?

As I've said in my first blog post, one of the main reasons I went with Godot in the first place was because I got fed up with downloading 10GB of "engine" before I could finally start developing my game.
Another important factor was that Godot Engine is open-source and its code is readable by mere mortals like me.
And finally it felt snappy and lean like some other editor based engines, without compromising in power.

* Small installation
* Open source
* Readable codebase
* Snappy and powerful

## Is plenty enough?

Short answer, yes.

Long answer, probably.
As I went along and my game started to take shape I couldn't help but look again to "competing" engines like Unreal Engine and Unity.
Longing for that feature that was missing in Godot that I desperately needed.

* Terrain
  * Unreal Engine 4 has [Landscapes](https://docs.unrealengine.com/4.27/en-US/BuildingWorlds/Landscape/Creation/)
  * Godot has [Zylann/godot_heightmap_plugin](https://github.com/Zylann/godot_heightmap_plugin)
* Behavior Tree
  * Unreal Engine 4 has [Behavior Tree](https://docs.unrealengine.com/4.27/en-US/InteractiveExperiences/ArtificialIntelligence/BehaviorTrees/BehaviorTreesOverview/)
  * Godot has [kagenash1/godot-behavior-tree](https://github.com/kagenash1/godot-behavior-tree)
* Level Streaming
  * Unreal Engine 4 has [Level Streaming](https://docs.unrealengine.com/4.27/en-US/BuildingWorlds/LevelStreaming/)
  * Godot has [an issue talking about it](https://github.com/godotengine/godot-proposals/issues/1197)

It ended up not being nearly as bad as I feared.
I often later realized I **(a)** didn't desperately needed it, **(b)** could make a simpler version of my own, or better yet **(c)** there was a plugin already available by someone else!
Also often in that order.

While Unreal Engine 4 has so many features built in, Unity has a much larger community to offer extensions, most things are available in Godot as well in one way or another.
And in my case (and I think many amateur/indie game developers), <abbr title="You aren't gonna need it">YAGNI</abbr>.

## Can't I program in X?

Short answer, yes.

Long answer, most likely. One thing that bugged me a little bit in the beginning is the fact that I had to use `GDScript`, Godot's own programming language to develop my game in.
Don't get me wrong, I love learning new programming languages, but what bugged me was that it didn't have a mature ecosystem like many other languages have.

Now 11.717 lines of `GDScript` I still feel it doesn't have a mature ecosystem, but I am amazed by how well it works!
Development in `GDScript` is extremely fast, and it contains just enough features (such as strong-typing) to make me feel comfortable.

Technically Godot supports any language through `GDNative` but that requires bindings contributed by the community, with limited support.
One "official" alternative is C# which requires a special mono build, I haven't given this a serious look because C# is one of my least favourite language/ecosystem.
Another one is C++, for which the Godot maintainers maintain the bindings for, I'm eyeing this one every so often but haven't made the leap yet.

## Would you go Godot again?

Short answer, yes!

Long answer, also yes. So far I have no real reason to use anything else than Godot Engine for any other games that I might be developing in the future.
Since my interest lie in retro games I have very little interest in none of them fancy features in Unreal Engine 4 or Unity.
I love having an editor available from the start, so there is very little chance I will ever go back to using engines like `libGDX`, although I do sometimes miss the control engines like that give me.

If one were to build a current generation 3D game that needed all the modern features, and engine like Unreal Engine 4 might be more suitable.
Purely because of sheer maturity.
But Godot is very capable as well, I reckon.

If one were to build a 2D game of any complexity, I would probably go for Godot.
It's effective, it's lean and offers all the features one could ever want in 2D.

## Conclusion

Godot Engine is great, go use Godot Engine.
It's great.
