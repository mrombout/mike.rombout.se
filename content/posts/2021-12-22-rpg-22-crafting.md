---
title: "RPG #22 - Crafting"
date: 2021-12-22T17:29:00+00:00
slug: rpg-22-crafting
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

Crafting is very popular in many games these days, and it can't be missing from mine either.

## Implementation

I wanted a simple, but re-usable system so that I could implement several types of crafting with the same system (e.g. smithing, tailoring, brewing, etc).
I managed to do with with `Recipes` and `Ingredients`.

A `Recipe` is defined using the following format, and defined which `ingredients` (simply other item ids) are required to create the item with the id defined in `item_id`:

```json
{
	"id": "iron_dagger",
	"name": "Iron Dagger",
	"skill_id": "smithing",
	"skill_level": 1,
	"item_id": "iron_dagger",
	"ingredients": [
		"iron_ingot",
		"iron_ingot",
	],
	"labels": ["smithing"]
}
```

When crafting an item we simply check if the player has the required skills, and if the container (usually the players inventory) has the right number of items.
If so, we remove all the ingredients from that container used by the recipe and add the crafted item.

```gdscript
func craft_item(container_id: String, recipe_id: String) -> int:
	if not _has_required_skill(recipe_id):
		return ERR_INVALID_SKILL
	
	if not _has_required_ingredients(container_id, recipe_id):
		return ERR_MISSING_INGREDIENTS
	
	var recipe = RecipeDatabase.find_recipe(recipe_id)
	var new_item_instance = ItemDatabase.create_item(recipe.item_id)
	InventoryManager.add_item(container_id, new_item_instance)
	
	var ingredients = recipe.ingredients.duplicate()
	for item_instance in InventoryManager.get_item_instances(container_id):
		if item_instance.item.id in ingredients:
			InventoryManager.remove_item(container_id, item_instance.id)
			ingredients.erase(item_instance.item.id)
	
	return ERR_NONE
```

And that's all there is to it for now, really.


## Reusability

It's a simple system, but implemented in a generic way which allows me to now quickly implement different kinds of crafting.
With just a bit of scripting and a custom GUI a lot of different crafting systems can be implemented.

* Smelting
  * Requires placing the items in the smelter (container) to smelt the ores into ingots.
* Smithing
  * Simply requires resources in the players inventory.
* Stronghold
  * A player-owned stronghold that automatically generates resources (in a hidden container). Once the Stronghold has generated enough resources the player can build expansions.

But for now I've also implemented a generic GUI that shows all recipes with a certain label.
This way I can quickly prototype new crafting systems without having to write a lot of code.

### Labels

I've started to use labels in most new systems I introduce, they're a great way to quickly filter out recipes for different use-cases.
E.g. mark all smithing recipes with the `smithing` label.
Or for example mark all cooking recipes that need an over with both the `cooking` label, and the `oven` label.

Then all there is too it, is to implement the GUI or action in such as way that it uses these labels how they are supposed to be used.

### Containers

When crafting an item a container must hold all the item instances that are used during crafting.
This way different types of crafting system can be implemented where items are in the players inventory, items must be added into another contained (e.g. a smelter or oven), or Stronghold expansions that hold their resources in a separate container.
