---
title: "RPG #16 - Scripted puzzle"
date: 2021-09-12T00:25:12+02:00
slug: rpg-16-scripted-puzzle
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

An old-school RPG is not complete without a bit of puzzling.
Whether we're talking puzzles in a more traditional sense, or those that are a bit less obvious.
What they all have in common is that they need to be implemented, and I want to do so using scripting to implement some truly unique puzzles.

As a first test to see if my scripting system (implemented in [RPG #6 - Scripting]({{< relref path="2020-10-20-rpg-6-scripting.md" >}})) is up to the task, I want to implement a common puzzle where the player is expected to put some stones in the correct position by rotating them.
Some might recognize this as the Spinning Symbol/Rune Tokens puzzles in Skyrim, but then without the eagles, snakes and the whales.

<video width="100%" autoplay loop muted>
    <source src="/vid/door_puzzle.webm" type="video/webm" />
</video>

It's a simple puzzle, but demonstrates a way to make several objects appear to interact with each other.
Following is the script associated with each of the rotation stones, which are triggered by the buttons.

```gdscript
extends EntityScript

var properties: Dictionary

var tween: Tween

var current_value: int
var target_value: int

func _ready():
	call_deferred('_initialize_tween')
	current_value = int(properties.initial_value)
	target_value = int(properties.target_value)
	set('intro_cave_puzzle', 0)

func _initialize_tween():
	tween = Tween.new()
	tween.connect("tween_completed", self, '_on_tween_completed')
	add_child(tween)

# triggered by a corresponding button
# each use we tween to a new position, cycling 4 positions
func use():
	if !tween.is_active():
		Audio.play_se(load('res://assets/sfx/stone_slide.ogg'))
		
		current_value += 1
		if current_value > 4:
			current_value = 1
		
		# store the position in a variable so that we can read it later
		var varname = 'intro_cave_puzzle_%s' % [properties.targetname]
		set_variable(varname, current_value)
		
		var new_rotation = Vector3(get_parent().rotation.x, get_parent().rotation.y + PI / 2, get_parent().rotation.z)
		tween.interpolate_property(get_parent(), 'rotation', get_parent().rotation, new_rotation, 2, Tween.TRANS_LINEAR, Tween.EASE_IN_OUT)
		tween.start()

# after each tween completes,
# we check if the stones are in the correct position
func _on_tween_completed(object, key):
	var stone1 = get_variable('intro_cave_puzzle_stone1', 0)
	var stone2 = get_variable('intro_cave_puzzle_stone2', 0)
	var stone3 = get_variable('intro_cave_puzzle_stone3', 0)
	var stone4 = get_variable('intro_cave_puzzle_stone4', 0)
	
	var stone1_is_correct = stone1 == 2
	var stone2_is_correct = stone2 == 3
	var stone3_is_correct = stone3 == 4
	var stone4_is_correct = stone4 == 1
	
	if stone1_is_correct and stone2_is_correct and stone3_is_correct and stone4_is_correct:
		# all positions are correct, trigger our parent
        # this will cause the door to open
		get_parent().trigger()
```

As you can see, the buttons mainly communicate through triggers (provided by Qodot) and variables.
So far the scripting system has passed the first test, I will continue to try it out as I implement new dungeons.
