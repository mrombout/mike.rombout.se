---
title: "RPG #7 - Debug Console"
date: 2020-10-24T22:57:00+02:00
slug: rpg-7-debug-console
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

Godot continues to amaze me with the speed at which I am able to implement features.
Today I while working on the scripting system I thought it might be useful to have an in-game debug console to quickly read and modify game variables.
And within less then an hour I managed to slap something together that worked quite a lot like an in-game console!

## Implementation

First I created a `Console` script which I added to the AutoLoad list of the project.
Then whenever `Console` is loaded, it adds a `CanvasLayer` with my new `ConsoleScreen` scene to the tree.
This way, whichever scene a I open I have access to the console window.

In this console window scene I've used a RichTextLabel for any console output and a LineEdit for gathering the input.
Whenever a command is entered the `Console.run_command()` method is called which will parse the command, run the command and emit a signal with output.
The console window will listen to the `output` signal and print whatever is coming in onto the screen.

![Console node tree showing a RichTextLabel used for output and a LineEdit for input](/img/debug_console_tree.png)

Command itself are simply registered using `Console.add_command(command_name: String, target: Object, method: String)` and for now all just live in the `Console` script itself.

![Demonstration of the help command to show all command and the get/set commands to modify variables](/img/debug_console.gif)

Now I have a framework to quickly interact with the game during development.
I can already think of some useful commands such as `godmode` to toggle god mode, `add_item` to add items to my inventory, `teleport` to teleport to different regions in the game.
