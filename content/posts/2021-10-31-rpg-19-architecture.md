---
title: "RPG #19 - Architecture so far"
date: 2021-10-31T18:00:00+01:00
slug: rpg-19-architecture-so-far
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

As the core features of my RPG are nearing completion (I keep telling myself that), I thought it might be a good idea to review the architecture of the game.

## Layers

{{<mermaid>}}
flowchart TB
  subgraph Module
    module
  end
  Module-->Game
  subgraph Game
    game
  end
  Game-->Engine
  subgraph Engine
    engine
  end
  Engine-->Godot
  subgraph Godot
    godot
  end
{{</mermaid>}}

In general all of the code in the RPG can be decided into the following layers.

**Module**
: Contains all assets, data and scripts that make up the majority of the content of the game. More than one modules can be loaded at a time, but at the moment there is only one.

**Game**
: Contains all scenes that and code that makes the game what it is.
These are things like the screens, player entities, camera control, etc.
Nothing in this layer is built for re-usability.

**Engine**
: Contains all low-level data management of the game.
It controls, stores and operates on all the data in the game and communicates what's going on to the **Game** by events.
Everything in this layer should be built for re-usability as much as possible.

The main idea between the split between **Game** and **Engine** is to be able to use large portions of the **Engine** in another game that I'm working on.
This game is an FPS instead of an RPG, it plays vastly different but uses many of the features I've built for the RPG.

## Engine components

The components in **Engine** are highly re-usable and take special care to run a independently as possible.

Each **Engine** component could be seen as a microservice.
They each handle their own bounded-context and handle their own storage.
Communication is done through events to avoid hard-coupling as much as possible, and calling other modules it kept to a minimum.

At the moment there are 28 modules, and their dependencies are as shown below.

{{<mermaid>}}
flowchart LR
  achievements;
  actions-->logging;
  actors;
  ai;
  audio;
  buff;
  console-->game;
  console-->scripting;
  console-->items;
  console-->options;
  console-->world;
  console-->gui;
  console-->quest;
  console-->skills;
  console-->events;
  console-->conversation;
  conversation;
  events;
  events-->scripting;
  game;
  game-->options;
  game-->world;
  game-->modules;
  game-->skills;
  graphics;
  gui;
  hud;
  items;
  journal;
  logging;
  modules;
  options;
  particles;
  party;
  pawns;
  player;
  procedural;
  qodot;
  quest;
  scripting;
  skills;
  world;
{{</mermaid>}}

Code in the **Engine** layer never calls code in the **Game** layer, instead they send events whenever something interesting happens in the engine code.

For example, when a new items has been added to the players inventory the `items` component sends a `item_added(item_id)` event.
The `narration` component in the **Game** layer listens to this event and shows a text notifying the player they've got a new item.
Additionally, if the player has their inventory screen open, the inventory screen will also listen to this event and automatically refresh the screen to contain the new item.

So far this approach have been working quite well.

* It's very easy to separate each concern into its own component.
* It's very easy to test each component as no graphics are needed.
* It's very easy to port to another language later (e.g. C++ or Rust) should I wish to do so.

## Game components

Code in the **Game** layer are not built for re-usability or testability.
They contain code making heavy use of Godot features and graphical capabilities.

In general it contains some game specific scenes such as the `Player` scene (consisting though of re-usable engine components such as the `FPSController`), and a lot of GUIs, HUDs and screens.

The screens and their GUIs call code in the **Engine** layer and listen to events coming from it.

I've found code in this layer to be a bit prone to errors since they are a bit more difficult to automatically test since they use graphical features.
E.g. testing if jumping works it not straight forward.

## Module components

The **Module** layer generally consists of scripts, rather than proper code.
These scripts contribute data such as items, quests, maps, etc to the game.
All scripts are completely impossible to test since they rely on run-time availability of entities and other scripts.
They are written in a very ad-hoc manner and very little good coding practices go into these because they are (by-design) very linear.

At the moment I haven't gotten far enough into writing content for the game, but I am reluctant to put any more engineering into the scripts as I want to avoid turning the game into a bland fetch X, kill X type of MMORPG or Ubisoft game (looking at you Far Cry and Assassin Creed).

# Conclusion

So far I'm the architecture haven't given me much grief.
Occasionally I've broken some code in the **Game** or **Module** layer by making a breaking change in an **Engine** component.
I trying to avoid this by writing strict tests on the **Engine** code that breaks whenever I make a incompatible change.
This at least gives me a hint that I should probably check any code using it.

I want to look into writing some tests for the **Game** code as well, where possible.
Even just loading each component up to see if they still load and run could be useful.

And finally I might want to merge some **Engine** components as there are some with overlap, or are simply too small to warrant their own component.
I've already did this in the passed with e.g. the `skills`, `attributes` and `motives` modules which now all live in `attributes`.