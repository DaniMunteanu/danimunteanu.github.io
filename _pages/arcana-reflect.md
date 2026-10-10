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

{% include video id="JveU_W238AE" provider="youtube" %}

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

<iframe frameborder="0" src="https://itch.io/embed-upload/19148763?color=ffffff" allowfullscreen="" width="1280" height="720"><a href="https://danimunteanu.itch.io/tales-from-the-teahouse">Play Tales From the Teahouse on itch.io</a></iframe>