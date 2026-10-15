---
permalink: /projects/tales-from-the-teahouse
toc: true
toc_sticky: true
toc_label: "Table of Contents"
toc_icon: "cog"
layout: single
author_profile: true
author: Daniel Munteanu
---

# Tales from the Teahouse

## Short Description

This project can also be found on 
<a href="https://github.com/DaniMunteanu/EudresInternship" target="_blank" rel="noopener noreferrer">Github</a> 
and 
<a href="https://danimunteanu.itch.io/tales-from-the-teahouse" target="_blank" rel="noopener noreferrer">itch.io</a> 

## Gameplay Video

{% include video id="o2s75bFVd6w" provider="youtube" %}

## Playable Web Build

<div class="itchio-widget-wrapper">
    <iframe frameborder="0" src="https://itch.io/embed/4857904?bg_color=141010&amp;fg_color=ffffff&amp;link_color=f21368&amp;border_color=5d5d5d" width="552" height="167"><a href="https://danimunteanu.itch.io/tales-from-the-teahouse">Tales From the Teahouse by DaniMunteanu, Bogdaain, 1Fearzz6, ruediuqqq, straw-berizz</a></iframe>
</div>

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