---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Introduction

## Building a 2D Game Engine

Welcome to this comprehensive guide on creating a 2D game engine from scratch using SFML and C++23, culminating in a fully functional platformer game! This tutorial will take you through the process of designing and implementing the core components of a game engine, from rendering and input handling to physics and state management.

We'll leverage the power of SFML for window management, graphics rendering, and audio while exploring modern C++23 features to build a robust and efficient engine. By following along, you'll gain a good understanding of game engine architecture and learn how to apply these principles to your own projects.

This guide aims to provide a practical and hands-on learning experience. Each chapter will build upon the previous one, gradually introducing new concepts and features. By the end of this tutorial, you'll have a solid foundation in 2D game development and a working platformer game to showcase your skills.

### Prerequisites

Before diving into the engine development, ensure you have the following prerequisites in place:

* **C++ Compiler:** A recent C++23 compatible compiler (e.g., Clang, GCC, MSVC). Clang 17 is used throughout the guide.
* **CMake:** For cross-platform build automation.
* **Clang-Format (optional but strongly suggested):** A tool that formats the C++ source code following a set of rules. I use the VSCode C++ extension that has the formatter built in.
* **Clang-Tidy (optional):** A static analysis tool that finds typical bugs and code smells.
* **Git (optional):** For version control and to access the provided code repositories.

### Structure

This tutorial is structured to be followed sequentially, as each chapter builds upon the concepts and code developed in the previous ones.

* Each chapter focuses on a specific aspect of the engine or game development.
* At the end of each chapter, you'll find a **Milestone** page, which contains a demo of the current state of the project.

There are two GitHub repositories associated with this tutorial:

* **Main Repository (Final Result):** [JP Engine](https://github.com/Penca53/jp-engine) - This repository contains the complete, final version of the game engine and the platformer game.
* **Milestones Repository:** [JP Engine Milestones](https://github.com/Penca53/jp-engine-guide) - This repository contains the code for each milestone, organized into separate directories. This allows you to track the progress and access the code at any stage of the tutorial.

The main repository contains a custom game engine and a sample 2D platformer game demonstrating its features. The engine provides foundational components for building 2D games, including scene management, rendering, basic physics, input handling, and resource management.

To begin your journey, proceed to [Project Setup](chapter-0/project-setup.md)!
