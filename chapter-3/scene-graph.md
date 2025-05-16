# Scene Graph

## What is a Scene Graph

A **Scene Graph** is a data structure, typically implemented as a **tree**, used to organize all the elements (objects, lights, cameras, etc.) within a 2D or 3D scene. It provides a powerful way to manage the spatial relationships, transformations, and logical grouping of objects within a complex game scene, simplifying its development.

* **Nodes:** Each element in the scene is represented as a 'node' in this graph.
* **Hierarchy:** Nodes are arranged in a parent-child relationship. A node can have one parent (except for the root node, which has none) and multiple children.
* **Transformations:** Crucially, each node usually stores spatial information like its position, rotation, and scale _relative_ to its parent node. The 'root' node represents the world's origin.

A concrete example may be a game character:

* The main body might be a node.
* The upper arm could be a child node of the body. Its position/rotation is relative to the body's shoulder joint.
* The lower arm could be a child node of the upper arm. Its position/rotation is relative to the elbow joint.
* The hand could be a child node of the lower arm, relative to the wrist.

## Why a Scene Graph

A scene graph provides a powerful and efficient way to manage the spatial relationships, transformations, and logical grouping of objects within a complex game scene, simplifying development and enabling performance optimizations.

* **Hierarchical Transformations:** This is the most fundamental reason. When you move, rotate, or scale a parent node, all its children (and their children, and so on) automatically inherit that transformation, maintaining their relative positions. Moving the character's body automatically moves the attached arms and hands correctly. This drastically simplifies animation and object placement.
* **Logical Organization:** It provides a structured way to group related objects. You can group all parts of a car, all furniture in a room, or all visual effects attached to a character. This makes complex scenes much easier for developers to manage, understand, and debug.
* **Agnostic Traversal:** The Node functions as an abstraction layer, separating the engine from the game's specific scene implementation. This enables the engine to traverse the scene without knowing the underlying details.
