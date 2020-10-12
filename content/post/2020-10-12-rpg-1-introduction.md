---
title: "RPG #1 - Introduction"
date: 2020-10-12
slug: rpg-1-introduction
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

Old-school first-person computer role-playing games are awesome.
Alright, gameplay-wise most of them didn't really age well.
But they looked awesome, didn't they? Not really.

But I'm still going to try and build a new one, just as clunky, just as ugly as the real deal!
But probably not as complex.

![Screenshot of Might and Magic VI](/img/mm6.png)

My goal is to build a classic 3D role-playing game to the likes of [Might and Magic VI: The Mandate of Heaven](https://en.wikipedia.org/wiki/Might_and_Magic_VI:_The_Mandate_of_Heaven) (pictured in the screenshot above), [The Elder Scrolls II: Daggerfall](https://en.wikipedia.org/wiki/The_Elder_Scrolls_II:_Daggerfall), [Realms of Arkania: Shadows over Riva](https://en.wikipedia.org/wiki/Realms_of_Arkania:_Shadows_over_Riva), [Wizardry 8](https://en.wikipedia.org/wiki/Wizardry_8) using the [Godot game engine](https://godotengine.org/).

Is building an RPG a good way to learn Godot? Probably not. But bad ideas never stopped me!

## Assumptions

In essence RPGs do not necessarily have to be complex, especially if we take old-school RPGs as our reference.
Most of these old-school RPG are based on tabletop role-playing games, and in essence these games can be boiled down to just a bunch of numbers, a random number generator (RNG) and a narrative.
From then on it's just more RNG, stat checks and ways that manipulate those numbers like equipment, combat, crafting, etc.

what I suspect makes RPGs complex to implement is getting all these systems to integrate and work together.
And of course building a good narrative to hide the fact it's all just numbers.

![Screenshot of Realms of Arkania: Shadows over Riva](/img/sa3.png)

## Inspiration

In order to prepare I've already taken a look at RPG Maker MV which pretty much confirms my assumptions.

RPG Maker MV provides all the systems ready to go.
The user then only needs to change the data in the Database to change the characters, monsters, their stats, etc.
And finally the biggest task is to they create content like maps and events to build the narrative.

What amazes me of this system though is that essentially it works with a limited scripting system through events, a shared list of global variables that can only hold primitives.
Yet people create the most amazing games with it.
Sure, there is the possibility to program in JavaScript as well, but not everyone uses it.

Another example is The Elder Scrolls: Skyrim, which through it's modding system allows quite a peek into how the game actually works.
It has a scripting system and the majority of the game is handled through Quests and Dialogues running tiny bits of script.

I am of course ignoring things like collision detection, pathfinding, animations, asset pipelines, et cetera because these are not specific to RPGs.

## Implementation

I've chosen to use [Godot](https://godotengine.org/) since it seems most equivalent to UE4 and Unity in the sense that it provides a full editor but makes no compromises in terms of capability.
But unlike those engines, Godot is open-source and comes in a single ~60MB executable!

Over the coming weeks I will try and tackle each part of the game.
It's going to be written almost like a journal, but might include some interesting bits if you run into the same problems as I did.
