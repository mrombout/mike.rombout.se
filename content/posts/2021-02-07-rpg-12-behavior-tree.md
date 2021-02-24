---
title: "RPG #12 - NPC AI"
date: 2021-02-07T13:04:00+02:00
slug: rpg-12-npc-ai
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

In previous blogs I've explained [how I implemented pathfinding for NPCs]({{< relref path="2020-10-19-rpg-5-pathfinding.md">}}), and later I [implemented it again]({{< relref path="2020-11-07-rpg-10-pathfinding-again.md" >}}).
But to be fair that by itself is not enough to create very interesting or realistic NPCs.

## Realism vs. Practicallity

Pretty much every RPG has some form of artificial intelligence built into the game.
Here a game designer must strike a fine balance between realism and practicallity (and many other factors, I'm sure).

With realism I mean how believable the NPCs appears to be part of the world.
In other words, does she come across as a person living her life in the world they were created in.

With practicallity I mean how practical is it in combination with the gameplay and also how practical is it to implement on a technical level.
For example having a pirate roam the world in his pirate ship when the game does not support sea-travel for the player is not practical and a waste of time.

If I look at the RPGs that I know there are varying levels of realism regarding AI.

On one end you have games like Elder Scrolls: Daggerfall where some NPCs have basic AI and some have no AI at all.
In this game unimportant NPCs are just randomly generated peasants that move around the city randomly.
Important NPCs such as those who give out quests stay at their locations (e.g. Innkeepers) or appear at certain locations at certain times (e.g. Lady Brisienna).

On the other end you have games like Skyrim which go to a certain length to have NPC go to work, have something to eat and finally to go back to sleep at the end of the day.
Here the engine takes care to also physically move the NPC towards their goal causing a lively atmosphere in certain places.

Despite the major different approaches I think both games manage to achieve a believable world in some way.

In Daggerfall the random peasants create the atmosphere of a busy city.
The clever placement of NPCs as well as ocassionaly moving these NPCs during the plot adds to the illusion that NPCs are not mere statues.

In Skyrim the NPCs move about their business which give the NPC more substance that they had in Daggerfall.
But if you start following them around you would notice they don't really do many interesting things during the day.
Here you can clearly see the designers chose a balance between realism and practicallity by only implementing major events such as day-time activities and sleeping.

## Implementation

As far as NPC AI goes I don't want to go all out and create complicated AI like in Bethesda's later Elder Scrolls series where NPC may for example grab something to eat or go to bed at certain times.
Instead I want to start with something simpler, something more appropriate for the era that I am trying to emulate with my RPG, something more akin to Bethesda's earlier games in the Elder Scrolls series.

And to be fair, many modern RPGs like for example Pillars of Eternity still follow this approach where NPCs don't really do much.

A common way to build AI that I am already familiar with is to use a Finite State Machine.
I also know that these can get out of hand when they become more and more complex.
But I also wanted to experiment with some other techniques to see if they could have any benefit.

### Finite State Machine

I think Finite State Machines are one of the most important in game development to keep things managable.
I've used them for split of large elements such as game state (`main_menu`, `gameplay`, `pause`, etc) to very tiny elements in a game such as turrets and enemies.

Since I'm already familiar with this approach I am confident that it will work, but I also know that it can become unmanagable the more states you introduce.
Using big, large states dedicated for one specific enemy keeps the FSM simple, but makes it hard to re-use states.
Building re-usable states makes the FSM grow significantly and makes maintaining if more difficult.

{{<mermaid>}}
stateDiagram-v2
    [*] --> Wandering

    Wandering --> Wandering: player not in sight
    Wandering --> Attacking: offended by player
    Attacking --> Wandering: player not in sight
    Attacking --> Fleeing: NPC can't win
    Fleeing --> Wandering: player not in sight
{{</mermaid>}}

In the example above I've implement the FSM of an NPC that randomly walks around in the `Wandering` state.
When the NPC is `offended by player`, it will go in the `Attacking` state in which it will follow the player around and tries to hit the player with their weapon.
Finally, when they realize `NPC can't win` they will flee and run in the opposite the direction.
In case `player not in sight` they will go back to `Wandering`.

I went with fairly high-level states for this one where the state implementation does a lot of work in code.
Some code like checking if the player is in sight is duplicated (but still abstract into a method) which makes things a little more difficult to maintain.

If I would go with this method I might investigate if Hierarchical State Machine might be a good fit to increase modularity.

### Behavior Trees

A behaviour tree is a tree in which one declares a series of task to be performed by the NPC where intelligence is added by the use of control flow nodes.
The main benefit of this approach appears to be the modular fashion in which you can build complex task out of many simpler tasks.
And if the implementation allows, you can re-use these tasks across NPCs.

It consists of only two type of node archetypes, **action nodes** and **control flow nodes**.

**Action Node**
: An action node represents a single task that can be performed by your NPC these may be things to do, or things to check.
: **Task Node**
  : A thing to do such as opening a door, picking up and item or swinging with a weapon. The "opening a door" task may fail when a door is locked.
: **Condition Node**
  : A thing to check such as, is the door open?, can I reach an item?, do I have a weapon?. These nodes succeed or fail based on whether the condition is true.

**Control Flow Node**
: As suggested by the name, these control the flow of the execution. You could compare these to control flow statements in programming such as `if` and `for`.
: **Selector Node**
  : Loops and executes each child node from left to right until one of them succeeds. You could compare this with a `for` loop that `break`s when a child node succeeds.
: **Control Flow Node**
  : Loops and executes each child node from left ro right until one of them fails. You could compare this with a `for` loop that `break`s when a child node fails.

In the most naive implementation (AKA *my implementation*) the BTREE is evaluated every tick in a depth-first fashion following the behaviour that I described above.

{{<mermaid>}}
graph TD
    BTREE --> selector1[?];
    selector1 --> sequence1["~"];
    sequence1 --> condition1([isDead?]);
    sequence1 --> action1[layDead];
    selector1 --> sequence2["~"];
    selector1 --> condition2([isPlayerClose?]);
    selector1 --> selector2[?];
    selector2 --> action2[determinePoint];
    selector2 --> action3[moveToPoint];
    selector2 --> action4[stopMoving];
    selector2 --> action5[wait];
    selector2 --> action6[resetPoint];
{{</mermaid>}}

Using this approach I managed to cobble together friendly NPC that will randomly wander around until it is killed.
The `determinePoint` and `resetPoint` nodes are probably too low-level to belong in a BTREE, but I included because of limitations in my path-finding system.

{{<mermaid>}}
graph TD
    BTREE --> selector1[?];
    selector1 --> sequence1["~"];
    sequence1 --> condition1([isDead?]);
    sequence1 --> action1[layDead];
    selector1 --> sequence2["~"];
    sequence2 --> condition2([isPlayerClose?]);
    sequence2 --> action2[wait];
    sequence2 --> action3[attackPlayer];
    selector1 --> sequence3["~"];
    sequence3 --> condition3([isPlayerClose?]);
    sequence3 --> action4[moveToPlayer];
    selector1 --> sequence4["~"];
    sequence4 --> action5[wait];
{{</mermaid>}}

Using many of the same nodes I implemented an aggressive NPC that waits until the player is within range and than starts moving towards the player.
When they are close enough to hit with the weapon they carrying they will try and strike the player.
As before, when they are killed they will stop moving and lay dead.

In general I can see the appeal of this modular approach but my naive implementation does not scale very well.
If I would use this method I would probably investigate event-driven behaviour trees to make them more performant and predictable.
I would also look into a way to aid in debugging such trees, as I can imagine building complex BTREES might get messy when they start misbehaving.

### GOAP

Goal-Oriented Action Planning (GOAP) is a way of implementing AI without specifically implementing the way to get there like in the other two approaches I mentioned.
In GOAP each NPC has their own view on the world (expressed in conditions/requirements) and a list of actions they can perform along with their preconditions, outcome and cost.
Using this information the NPC will determine a plan actions themselves to reach a certain goal.

I did not implement this system myself, but instead played around with [RodZill4/godot-goap](https://github.com/RodZill4/godot-goap) on GitHub to see if this system would be valuable for my game.

While I do think using GOAP will create an interesting and lively world it does require a lot of work to implement and debug to avoid conflicts and strange behaviour.
I will certainly investigate this approach a little more once I have implemented all the other systems in my game.
Also while doing my research I stumbled upon [Utility systems](https://en.wikipedia.org/wiki/Utility_system) used by The Sims series of games, which solves the problem in a similar fashion as GOAP.

## Conclusion

It was fun to play around this several way to implement AI.
But given the amount of stuff I still have to implement in the game I've decided to go with the FSM mainly to reduce the scope of this system.

I would love to have interesting NPCs that go about their daily life on their own, but in reality most players probably won't notice.
Many games, even to this day have static NPCs that never move and use other means to create a believable world such as strong character personalities.
While creating those also takes a lot of time, at least I can procrastinate creating those for good conscience!

<sub><sub>at least that's what I tell myself</sub></sub>