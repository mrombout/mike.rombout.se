---
title: "RPG #6 - Scripting"
date: 2020-10-21T20:12:00+02:00
slug: rpg-6-scripting
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

When I started to build my RPG I argued that most RPGs are just a bunch of systems working together in a systematic way, and I so far I find it to be still true (for most games actually).
But another large component of any good RPG is story and that requires a whole lot of non-systematic interaction with the systems.

To me a good RPG successfully obscures their systems and dice-rolls by using interesting puzzles, cut-scenes, interactions and events.
And each and every one of these events and puzzles have to be unique in order to avoid being repetitive and breaking immersion.
I mean who wants to climb radio-towers for an entire games length.

These functionalities are usually not worth implementing in the game engine, and this is were scripting comes in.
Godot doesn't come with what I would consider a traditional embedded scripting system.
It has GDScript, but I disregarded that at first because I thought I would lack control over it (I was wrong).

## DSL

At first I started with what I know, building my own DSL.
I built a quick assembly like DSL and an interpreter in GDScript to control parts of the game.
Using a program counter I have full control over when I want the interpreter to play, and when I want it to wait.
This is useful when I want to wait for player interaction during a conversation for example.

```
start:
	narrate "You hear a buzzing coming from the tree"
	choice_menu "Investigate?"
	choice "Yes please!" yes
	choice "I'd rather not." no

yes:
	narrate "Not the bees! AAAAAHHHH! My eyes!"
	decrease player_health 10

no:
	narrate "You decide to leave it alone and live another day."
```

But I soon found that building a full DSL in GDScript is probably going to be quite labour intensive and hard to maintain due to lack of good parser generators and unit testing frameworks (there's [Gut](https://github.com/bitwes/Gut) though).
I would have to move to C/C++ for those sweet parser generators, but then I might as well use a binding to an existing scripting language like Lua.

## GDScript

After looking for a couple of different alternatives I realized Godot **does** come with a scripting language built-in, GDScript!
So far it had proven to be very powerful, so why not also use it for scripting?

The only problem that would need to be solved is to pause the script processing whenever I want (e.g. when waiting on UI interaction from the player).

### Threading

It took me 5 lines of code to implement a proof of concept for this.
I use a `Thread` to run the scripted code on so that the game doesn't freeze.
I use a `Semaphore` to pause the thread when a dialogue option shows up.
When the player clicks a dialogue option I `post()` to the Semaphore causing the thread to continue.

> Unfortunately Godot does not support debugging on a thread ([#2446](https://github.com/godotengine/godot/issues/2446)).

I played around this with and implemented a simple level that plays an animation and a sound and then sends a signal to a nearby gate which in turns also plays an animation and a sound, opens up and disables it's collision box to allow passage.

```gdscript
# EntityScript contains some helper methods commonly used in scripts. It's not
# required to extend it, but it is very helpful.
extends EntityScript

# We set our initial state based on the value of the `cell_door_open` variable. This
# way it will be able to persist its state.
func _ready():
	VariablesManager.connect('variable_changed', self, '_on_cell_door_open_changed')
	_update_state()

# Returns a list of actions that can be performed on this entity.
func actions():
	return [
		Action.new(tr('Use'), self, 'action_use')
	]

# Is executed whenever the `use` action is performed by the player. It will simply
# switch a variable and let the connected signal receiver to the rest.
func action_use():
	set_variable('cell_door_open', !get_variable('cell_door_open'))

# Whenever the `cell_door_open` variable changes we update our state.
func _on_cell_door_open_changed(name, value):
	if name == 'cell_door_open':
		_update_state(value)

# We update the state of our parent (the lever), which will play a sound and animation
# whenever it changes from `Up` to `Down`.
func _update_state(value):
	if get_variable('cell_door_open', false):
		get_parent().state = 'Up'
	else:
		get_parent().state = 'Down' 
```

So far it appears to be working reliably and I will continue to use it more when I start implementing the first real dungeon.
For now I will take a peek at RPGMaker and see if I can already implement some additional scripting functions that look useful.

![](/img/scripted_lever.gif)

### Alternative: Using `yield()`

A `yield()` could be used to "freeze" the state of a script until as explained in the docs [Coroutines & signals](https://docs.godotengine.org/en/stable/getting_started/scripting/gdscript/gdscript_basics.html#coroutines-signals).
I have already used `yield()` with signals several times, but I did not try it for the scripting yet since my other solutions works fine for now.
