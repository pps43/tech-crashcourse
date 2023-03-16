- [Godot](#godot)
  - [Core Design](#core-design)
    - [OOP rather than ECS](#oop-rather-than-ecs)
    - [Node based](#node-based)
  - [Scripting](#scripting)
    - [GDScript](#gdscript)
  - [Assets](#assets)
    - [File System](#file-system)
    - [Assets pipline](#assets-pipline)
  - [Physics](#physics)
  - [Networking](#networking)
  - [Animation](#animation)
  - [2D](#2d)


# Godot

Godot is a [completely free, opensource](https://github.com/godotengine/godot#free-open-source-and-community-driven) 2D/3D game engine made with C++ (Godot 4 is written in C++17, and [released on Mar. 1st 2023!](https://godotengine.org/article/godot-4-0-sets-sail/)). Godot is also capable of making non-game apps (e.g., [Pixelorama](https://github.com/Orama-Interactive/Pixelorama)), using `low-processor mode`.

Godot is super-crossplatform (can run editor in browser, and can run engine in server).

## Core Design

- OOP, rather than ECS.
- Node based.
- Editor itself is a Godot Game.
- Home-made tools all in one (especially for 2D).

### OOP rather than ECS

[Official Blog](https://godotengine.org/article/why-isnt-godot-ecs-based-game-engine/)

Godot heavily uses concept of `node` and `inheritance` because of simplicity, explicity, and flexibility, while in higher level, it supports composition. `node` is not as heavy as "gameobject" in Unity, but as lightweight as "component" in ECS.

Also you can implement ECS pattern using Godot, e.g., [Godex](https://github.com/GodotECS/godex)

### Node based

- A `game` is a tree of scenes.
- A `scene` is a reusable hierarchy of nodes, acting as "scene" and "prefab" in Unity's term. `scene` can be nested, or inherited. Think of a `scene` as a class.
- A `node` is smallest building blocks, each with a specific purpose. Most nodes work independently from one another, not like "component" in Unity's term. `nodes` emit signals when event occurs, and can bind to other nodes' function.


## Scripting

You can write logic in GDScript, C# (via Mono), GDNative, or C++ modules.

The reason why GDScript rather than Lua, Python, JS... is that:
- Poor threading support for them.
- Poor performance for them  to adapt to Godot OOP design.
- Poor binding to C++ for them.
- No native type on vector/matrix.
- Heavy GC memory usage.
- Difficult integrating with Editor.

`GDScript` is a custom scripting language for Godot, to achieve high performance and easier interaction with the engine. [Here](https://docs.godotengine.org/en/stable/about/faq.html#what-were-the-motivations-behind-creating-gdscript) are more benefits.

`GDNative` gives you access to most of the API available to `GDScript` and `C#`, with higher performance.

`GDNative` is not limited to `C/C++`. With third-party bindings, actually you can use it with other languages (`Python`, [`Rust`](https://github.com/godot-rust/gdnative), etc).

The differences betwwen GDNative and C++ modules are listed [here](https://docs.godotengine.org/en/stable/tutorials/scripting/gdnative/what_is_gdnative.html#differences-between-gdnative-and-c-modules).

### GDScript

- Object-oriented, gradual-typed (e.g., Typescript). with Python-like Grammar. [Discussions](https://github.com/godotengine/godot-proposals/issues/6031) on improving its performance.
- Every GDScript file (`.gd`) is implicitly a class.
- Built-in [engine event callback](https://docs.godotengine.org/en/stable/tutorials/best_practices/godot_notifications.html): `_ready()`, `_process(delta)`, `_physics_process(delta)`, `_draw()`, etc.
- Use `export` before `var` to show in inspector
- Use `$xxx` as shortcut for `get_node("xxx")`

## Assets 

### File System

[doc](https://docs.godotengine.org/en/stable/tutorials/scripting/filesystem.html#doc-filesystem)

- `res://`
- `user://`
- No folder naming conventions.
- No file metadata, nor asset database.
- CRUD all assets within Godot, or dependencies will break.

### Assets pipline
[doc](https://docs.godotengine.org/en/stable/tutorials/assets_pipeline/import_process.html)

## Physics

Godot used to use `Bullet` engine for 3D, but Godot4 begins to use  its in-house `Godot Physics`.


## Networking


## Animation

- In-house Tween system

## 2D

- Level-editing tools. Tilemap.
- 2D light & shadow.