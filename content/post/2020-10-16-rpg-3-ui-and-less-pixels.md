---
title: "RPG #3 - UI and viewports"
date: 2020-10-16
slug: rpg-3-ui-and-viewports
categories:
  - Dev Blogs
tags:
  - games
  - rpg
  - godot
---

Millions of pixels, millions of colours. Who needs all those? Not me!

Pixel-art sprites are still massively popular in games.
And many older games that looked good back then, would still be considered to look good today.

Unfortunately old-school 3D graphics rarely get the same love and sometimes are even considered ugly. Well, they are, and they're great!

## Implementation

### Viewports

Many older 3D RPGs render only a small portion of the screen in 3D as you can see in this screenshot of Realms of Arkania III below. This is most likely also a way to cut down the rendering cost and improve performance.

![Screenshot of Realms of Arkania showing that only a portion of the screen is 3D rendered](/img/sa3.png)

In Godot this can be implemented by using a `Viewport` and a `Sprite` that contains a `ViewportTexture`.
Then we can make this sprite smaller than the game window and move in to where we want in the GUI.
We could even hide it behind ugly curtains like in Realms of Arkania.

![SceneTree showing a viewport and a sprite rendering the viewport to a texture](/img/render_to_texture.png)

Here `main_viewport` contains the rendered game view and `main` contains a `ViewPortTexture` targeting the `main_viewport`.

### Low Resolution

Currently the resolution is still too high.
Since we are already rendering to a `ViewportTexture` this turned out to be quite easy to reduce the resolution.
I changed the main viewport to be a size of 240x240, then I scaled the rendering sprite to 480x480 and turned the `Filter` flag off. 

The effect is especially visible in distant terrain.

![Difference between 1x render vs 2x render](/img/pixely_pixels.gif)

#### Alternative

I could have just made the entire game resolution smaller and enable stretch in Godot's project settings to `Mode: 2d`, `Aspect: keep` but I figured I might need the extra bit of space later for the UI.

Also by using the `ViewportTexture` I can make the 3D an even worse resolution if I wanted to by rendering for example at 120x120.

### Minimap

Since I figured out how to render to texture I created a very simple minimap.
The technique is similar in that it uses a viewport and a sprite to render to.
But instead of rendering the main camera's view I render the view of an Orthogonal camera that follows the player around in a separate viewport.

Obviously this technique won't work for dungeons, moving under things or clearly making things out such as NPCs.
But at least it forms a good placeholder for that hole I made in the GUI for the minimap.
