---
title: "RPG #27 - Inspector Fields"
date: 2022-08-31T20:55:00+20:00
slug: rpg-27-inspector fields
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

In [RPG #26 - Objects dock]({{< relref path="2022-05-24-rpg-26-objects-dock.md" >}}) I've implemented a dock to show all resources of the game.
Having that implemented already gives me access to Godot's Inspector which allows me to edit any selected resource.
This saves me some time having to write custom forms for each resource.

In order to make the inspector a little more useful I decided to implement a couple of generic custom editors for the Inspector.

I've followed [Inspector plugins](https://docs.godotengine.org/en/stable/tutorials/plugins/editor/inspector_plugins.html) to do the first implementation.

## ValidatedLineEdit

First I created the `ValidatedLineEdit` which is generic editor using a `LineEdit` with the additional feature that it can validate its current value.

```gdscript
extends EditorProperty

const ErrorFieldStyle = preload('res://addons/dod_editor/inspector_editors/error_field.tres')

var line_edit_control := LineEdit.new()
var current_value := ""
var updating := false

func _init():
	add_child(line_edit_control)
	add_focusable(line_edit_control)
	line_edit_control.text = current_value
	line_edit_control.connect("text_changed", self, "_on_text_changed")

func _on_text_changed(new_text: String) -> void:
	# Ignore the signal if the property is currently being updated.
	if updating:
		return

	# Set the current value
	current_value = new_text
	emit_changed(get_edited_property(), current_value)
	
	# Validate the current value
	if not _is_valid(current_value):
		line_edit_control.add_stylebox_override('focus', ErrorFieldStyle)
		line_edit_control.add_stylebox_override('normal', ErrorFieldStyle)
	else:
		line_edit_control.add_stylebox_override('focus', null)
		line_edit_control.add_stylebox_override('normal', null)

func update_property():
	# Read the current value from the property.
	var new_value = get_edited_object()[get_edited_property()]
	if (new_value == current_value):
		return

	# Update the control with the new value.
	updating = true
	current_value = new_value
	line_edit_control.text = str(current_value)
	updating = false

func _is_valid(current_value: String) -> bool:
	return true
```

This custom editor will call `_is_valid()` every time the value is changed.
This function is designed to be overriden by inherited editors to do some actual verification.

<a href="/img/inspector_validated_fields.png">![Example of a validated field](/img/inspector_validated_fields.png)</a>

## RegExValidatedLineEdit

I then implemented a flexible validated editor that validates field values according to a regular expression.
The value is only considered valid when it matches the regular expression.

```gdscript
extends "res://addons/dod_editor/inspector_editors/ValidatedLineEdit.gd"

var compiled_regular_expression: RegEx

func _init(regular_expression: String) -> void:
	compiled_regular_expression = RegEx.new()
	compiled_regular_expression.compile(regular_expression)

func _is_valid(current_value: String) -> bool:
	return compiled_regular_expression.search(current_value) != null
```

## LoadedModuleResourceValidatedLineEdit

An editor more specific to my game is the `LoadedModuleResourceValidatedLineEdit`.
This editor considers the value as valid if the current value matches with the `resource_field` of any resource of the type `resource_type`.
For example, a field could validate if the value matches with the `id` field of a loaded `map` resource.
This is very useful for e.g. `markers` which refer to a `map` by their `id` field.

```gdscript
extends "res://addons/dod_editor/inspector_editors/ValidatedLineEdit.gd"

var plugin: EditorPlugin
var resource_type: String
var resource_field: String

func _init(plugin: EditorPlugin, resource_type: String, resource_field: String) -> void:
	self.plugin = plugin
	self.resource_type = resource_type
	self.resource_field = resource_field

func _is_valid(current_value: String) -> bool:
	var loaded_modules = plugin.objects_dock.loaded_modules
	for loaded_module_id in loaded_modules:
		var loaded_module = loaded_modules[loaded_module_id]
		var resources = loaded_module.get(resource_type)
		if resources:
			for resource in resources:
				var resource_field_value = resource.get(resource_field)
				if resource_field_value == current_value:
					return true
	return false
```

## Conclusion

Implementing a custom [`EditorProperty`](https://docs.godotengine.org/en/stable/classes/class_editorproperty.html#class-editorproperty) was quite simple.
While the new editors that I've introduced are quite simple, they make sure that I can only enter valid values.

Now that I know how to implemented custom editors also opens up the possibility to implement more advanced ones.
One idea that I'm keen to try is implementing a custom editor for container items for example.