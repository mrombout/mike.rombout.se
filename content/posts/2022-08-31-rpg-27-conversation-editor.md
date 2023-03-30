---
title: "RPG #27 - Conversation Editor"
date: 2022-08-30T18:34:00+00:00
slug: rpg-27-conversation-editor
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

Any modern games needs dialogue these days.
This is especially true for RPG because building a believable world is much easier if you can fill in with beliavable and lovable (or hatable) characters.

Godot doesn't come with anything built-in, but there are plenty of addons to choose from like [godot-ink](https://github.com/paulloz/godot-ink), [dialogic](https://github.com/coppolaemilio/dialogic), [godot_dialogue_manager](https://github.com/nathanhoad/godot_dialogue_manager) and [whiskers](https://github.com/LittleMouseGames/whiskers) to name a few.
But none of these quite fit my needs, but my biggest concern as usual with external addons is that there's no guarantee they will be maintained for any decent amount of time.

So with the thought of having to maintain it myself anyway, I decided to build yet another dialogue system from scratch.
With the added benefit of being built specifically for my game, which might come in handy.

## Previous Implementation

In [RPG #13 - Big update of small updates]({{< relref path="2021-04-29-rpg-13-big-update.md" >}}) I very briefly mentioned that conversations with NPC has been implemented.
It was very easy to do so, because it was essentially just using the `ScriptingEngine` that I had already built before.

A conversation was represented using a special `gdscript`:

```gdscript
extends ConversationScript

func topics() -> Array:
  return [
    "silvercreek",
    "edwin"
  ]

func ask_silvercreek() -> String:
  reveal_topic("edwin")
  return "Lorum ipsum dolor sit amet."

func ask_edwin() -> String:
  return "Consectetur adipiscing elit."
```

It allowed for very simple conversation, without much interaction in the form of choices.
Also as conversation grew longer it was very difficult to keep track of where the conversation was going.

## New Implementation

After studying dialogue in other game engine I decided to make it largely data-driven system, enhanced with some evaluated expressions and scripts.
This seems to work for games such as [Divinity](https://docs.larian.game/Dialog_editor) and [Skyrim](https://www.creationkit.com/index.php?title=Dialogue_Views_Tab), so it must work fine for my case.

In an oversimplified illustration would look something like this:

{{<mermaid>}}
classDiagram

Dialogue o-- BaseNode
BaseNode <|-- Answer
BaseNode <|-- Question
BaseNode <|-- Topic
BaseNode -- "0..n" BaseNode : next
Answer -- Expression : condition

class Dialogue
class BaseNode{
    +id: String
}
class Topic
class Answer{
    +text: String
}
class Question{
    +text: String
}
class Expression

{{</mermaid>}}

A `Dialogue` class represents a dialogue between any amount of characters.
It is a graph of `BaseNode`, which represents all the different kind of nodes that a conversation can hold.
For example an `Answer`, a `Question`, a `Topic`, a `ChoiceMenu` you name it.

This makes up a model of the conversation which is then fed into a `DialogueInterpreter` which processes it step by steps and sends appropriate signals for the GUI to respond to.

It wasn't very difficult to implement.
At least now the user can be offered some choices during a conversation, but for a developer conversations are still difficult to develop.

## Editor

The biggest benefit of going with the model based approach is that it's now much easier to build a visual editor.
So that's what I did.

<a href="/img/conversation_editor.png" target="_blank">![Custom dialogue editor in Godot](/img/conversation_editor.png)</a>

Whenever you select a `.tres` resource that is a dialogue, the dialogue editor opens up.
In here the dialogue is presented using Godot's [`GraphEdit`](https://docs.godotengine.org/en/stable/classes/class_graphedit.html).

Nodes in a conversation and be added, moved and removed visually.
This gives a better overview on the flow of a conversation.

Using the preview window on the right a developer can preview a conversation from the start or the selected node.