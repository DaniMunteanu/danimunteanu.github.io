---
permalink: /projects/arcana-reflect
layout: no-sidebar
toc: true
toc_sticky: true
toc_label: "Table of Contents"
toc_icon: "cog"
author_profile: false
classes: no-sidebar
---

# Arcana Reflect

## Short Description

This game started as an entry for the 
<a href="https://rgda.ro/game-jam-universitar-2026/" target="_blank" rel="noopener noreferrer">2026 RGDA University Game Jam</a>.
I worked on this project along with other 5 students with which I've never collaborated before.
Despite this, development went smoothly and we even agreed to expand the game after the game jam ended.

The result is a **2D (and 3D!) puzzle game** built using the **Godot** engine.
The mechanics I implemented include:
- Drag & Drop 2D puzzles
- Inventory system
- Puzzle progression tracking system using a notebook UI

This project can also be found on 
<a href="https://github.com/DaniMunteanu/Game-Jam-Game" target="_blank" rel="noopener noreferrer">Github</a> 
and 
<a href="https://biesbi.itch.io/arcana-reflect" target="_blank" rel="noopener noreferrer">itch.io</a> 

## Gameplay Video

{% include video id="JveU_W238AE" provider="youtube" %}

## Download

<iframe frameborder="0" src="https://itch.io/embed/4511997" width="552" height="167"><a href="https://biesbi.itch.io/arcana-reflect">Arcana Reflect by BiesBi, DaniMunteanu, Natalia Rosu, Lluke18, Willo_Flow, RaluAndreee, Dev Wizards</a></iframe>

## Implemented Mechanics

```gdscript
func on_try_snapping(painting_index: int):
    var closest_marker_index = -1
    var closest_distance = snap_max_distance
    var painting_center = paintings[painting_index].global_position + paintings[painting_index].size / 2

    for marker_index in range(markers.size()):
        var dist = painting_center.distance_to(markers[marker_index].global_position)
        if dist < closest_distance:
            closest_distance = dist
            closest_marker_index = marker_index

    if closest_marker_index != -1:
        if paintings_snapped.has(closest_marker_index) and paintings_snapped[painting_index] != closest_marker_index:
            paintings_snapped[painting_index] = -1
            return
        paintings_snapped[painting_index] = closest_marker_index
        paintings[painting_index].global_position = markers[closest_marker_index].global_position - paintings[painting_index].size / 2
        #paintings[painting_index].global_position = markers[closest_marker_index].global_position
        if closest_marker_index == painting_index:
            TextManager.show_text(painting_descriptions[painting_index])
        check_if_solved()
    else:
        paintings_snapped[painting_index] = -1
```