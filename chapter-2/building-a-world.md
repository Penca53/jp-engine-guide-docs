# Building a World

## The Play Environment

The goal of the project is to make a simple game engine while developing a demo 2D platformer game alongside it.  When it comes to creating the 2D world for a platformer game, we can either painstakingly place individual objects such as platforms, obstacles, and enemies, or opt for a more organized system: a Tilemap.

## Mosaic World

Imagine a 2D game world like a big grid. Each individual cell in that grid is like a square on a checkerboard. In a Tilemap system, each of these grid cells can hold a **Tile**.

Essentially, a **Tile** is a small, often square, image or sprite that represents a visual element in your game world. Think of it as a single piece of a larger homogeneous mosaic. All the tiles are part of a container, a Tileset.

A **Tileset** is like the artist's palette for your **Tilemap**.

Imagine you have a collection of individual paint colors. A Tileset is a collection of all the different Tile images you can use to paint your game world.

It's a single image (can be multiple images, but in our engine, we'll constrain it to a single image) that contains multiple individual Tiles arranged in a grid. The game engine then knows how this image is organized and can pick out the specific Tile you want to use for a particular cell in your Tilemap.

The **Tilemap** is the canvas where you create your 2D game world using the Tiles from your Tileset.

Think of it as a structured grid that overlays your game scene. Each cell in this grid corresponds to a specific location in your game world. The Tilemap's job is to keep track of which Tile from your Tileset is placed in each of these grid cells and draw them.

In short:

* **Tile:** A single, small image representing a visual element.
* **Tileset:** A collection of these individual Tile images, organized for easy access.
* **Tilemap:** The grid structure that stores information about which Tile from the Tileset is placed at each location in the game world.
