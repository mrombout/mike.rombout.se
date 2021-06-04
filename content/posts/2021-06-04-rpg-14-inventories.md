---
title: "RPG #14 - Inventories"
date: 2021-06-01T19:58:00+02:00
slug: rpg-14-inventories
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

Over the last couple of weeks I'd been preoccupied with other things, with the main **thing** being moving to Sweden!
So I figured I might as well write up a blog post of one of the new features I talked about in [RPG #13 - Big update of small updates]({{< relref path="2021-04-29-rpg-13-big-update.md">}}), in particular the inventory system.

## Design

I originally designed the cleverly named `InventoryManager` system to manage the player's inventory.
But to cut down on the number of systems that I had, it eventually also took on the responsibility of managing all the items in the entire world.
And so far that it's working out really well, if I say so myself.

{{<mermaid>}}
classDiagram
    class ItemDatabase
    class InventoryManager
    class Inventory
    class Script

    InventoryManager "1" *-- "*" Inventory
    Inventory "1" *-- "*" ItemInstance
    Inventory "1" o-- "1" Script
    ItemInstance .. ItemDatabase
{{</mermaid>}}

Like all of the systems in my design, the `InventoryManager` acts as the facade to everything related to inventories.
From here I can manipulate each inventory by adding, removing, or transferring item instances based on their `instance id`.

An `Inventory` is just a collection of `ItemInstance`s designed for quick retrieval and mutation by `instance id` (a unique string, a `GUID` at the moment).
It also has the responsibility to trigger any scripting hooks and events to allow dynamic behavior of these inventories.

An `ItemInstance` is a unique instance of an `Item`, pointing to it by its `item id` in the `ItemDatabase`.
An `Item` defines an item's attributes such as id, name, slots, damage, etc.
An `ItemInstance` copies all these attributes and represents a unique instance of this `Item`.
Thus allowing these items to be e.g. customized by the player, maintain their own wear, etc.

So like many of the systems, it's essentially just a specialized data store.
It's very easy to interact with using scripting, and also very easy to persist and restore from disk.
Perhaps I should be looking into writing (or integrating) an embedded document store for all of this (but I'm getting ahead of myself).

This has proved to be quite flexible as I've used inventories for more than just the player inventory.

## Usage

Obviously, the player's inventory is implemented using this system.
For now, the player can only own one inventory, so I've just hardcoded the `player` inventory to be that of the player.

It's containers that are in the world such as chests, barrels, etc.
Having them available through the `InventoryManager` makes it easy to access them through scripting at any point in time.
Being able to put `Script`s on inventories makes it easy to e.g. trigger traps, generate random contents, etc.

![Screenshot of Draeywin showing the Spellbook using the inventory system to store the spells](/img/store.png)

Next, I'm using inventories for stores.
A store simply displays an inventory.
When a player sells or buys the items are simply transferred from one inventory to another.
Again a `Script` may be used to e.g. restock and store dynamically.

![Screenshot of Draeywin showing the Spellbook using the inventory system to store the spells](/img/spellbook.png)

Finally, and this is a bit of a hack, I'm using inventories for the player spellbook.
Since spells are something that the player can wield, they are simply implemented as items (weapon to be exact).
So rather than invent another system I've created a dedicated `players_spellbook` inventory that contains all the player's spells.
This inventory is only shown in the game through the spellbook screen.

## Conclusion

So far I'm quite pleased with how the `InventoryManager` works.
I don't expect any major changes in the overall design, except maybe the item prototyping system, simply because I haven't used this feature in the game yet.