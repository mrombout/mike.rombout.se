---
title: "RPG #18 - Attributes"
date: 2021-10-22T23:42:00+02:00
slug: rpg-18-attributes
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

Ah, we're back to the numbers again.
In particular, I've implemented `Attributes`, which is the system that I've used to manage most values belonging to an actor (e.g. the player).

An RPG has many numeric values that are important to the player.

Usually there are some **major attributes** like `Strenght` that determines for example how hard the player can hit, `Perception` to represent how perceptive the player is, etc.

Additionally, there could be **skills** which measure particular skills and crafts that the player can perform such as `Weaponsmithing`, `Weaving`, etc.

And sometimes there are also **hidden attributes**, these are not always directly shown to the player but very important for the experience.
For example `Damage` to measure the players damage, `Critital Rate` how effective/common critial hits are, `Defense Rating` how well the player is defended. These are often based on the other properties such as what the player is wearing, how strong they are, etc.

## Design

In my current design all of the values mentioned above are `Attributes`.
An `Attribute` is simply a value belonging to an actor, and it's backed by an `AttributeState` for each actor that owns such an attribute.

Additionally, each `Attribute` can be associated with one or more tags.
This makes it easier for me to treat these different attributes in different ways. For example all attributes with the `major` tag will be shown separetely from attributes with the `skill` tag.

## Implementation

Most of it is simple implemented using a straight-forward object-oriented approach.

### `ConstantAttribute(State)`

A `ConstantAttribute` is a simple numeric value that is stored in the player's save file.
These are generally used for **major attributes** since they are simple values.

### `ExpressionAttribute(State)`

An `ExpressionAttribute` is a special kind of calculated property that uses an expression to resolve it's value. These are great for **hidden attributes** since they can use the values of other attributes in a flexible manner.

```gdscript
ExpressionAttribute.new({
  'id': 'max_health',
  'name': 'Max. Health',
  'expression': '(strength + endurance) / 2',
  'dependencies': ['strength', 'endurance'],
  'show': true,
  'labels': ['dynamic'],
}),
```

### `DamageAttribute(State)`

A `DamageAttribute` is an example of how code could be used to create custom attributes.
The major benefit of using a custom attribute is that they could then also be used in `ExpressionAttribute`.
In this particular case I just implemented a custom attribute because of some legacy code I didn't move to the attributes system yet.

In the implementation I ask the `EquipmentManager` for the weapon rating of the `player` and add the players strength to it.

```gdscript
func value():
	return EquipmentManager.get_weapon_rating('player') + Attributes.get_attribute('player', 'strength').value()
```

## Conclusion

Using this system I've managed to implement all of the attributes that I've needed so far.
Since all of these actor related numeric values are now using one system and are defined in one place it allows me to design a generic buff/debuff system that can affect any attribute from `Weapon Damage` to `Carpentry`.
