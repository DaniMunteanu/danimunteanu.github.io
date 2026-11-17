---
permalink: /projects/dreamcatcher
layout: single
author_profile: true
author: Daniel Munteanu
toc: true
toc_sticky: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

# Dreamcatcher

## Short Description

I made this **2D pixel-art Godot project** as the subject of my Bachelor's thesis on **Rogue-like video games with procedurally generated levels**.

In the span of 7 months, I developed a **dungeon-crawler** video game, where the player can explore multiple rooms, fighting enemies in order to collect resources. Said resources can be used to purchase upgrades or HP points from a Shopkeeper NPC found in one of the rooms. The player's final objective is defeating the floor's boss.

This project was my first attempt at using Godot and drawing pixel-art assets, which is why the priority of this project was creating a single floor with all necessary systems. However, I plan to expand this game in the future, adding more floors, playable characters, enemies, bosses, and more impactful upgrades.

For now, these are the more interesting systems I have implemented:
- Procedural generation
- Save file system
- Shop and upgrade system

This project can also be found on 
<i class="fa-brands fa-github"></i>
<a href="https://github.com/DaniMunteanu/DreamCatcher" target="_blank" rel="noopener noreferrer">Github</a> 
and 
<i class="fa-brands fa-itch-io"></i>
<a href="https://danimunteanu.itch.io/dreamcatcher" target="_blank" rel="noopener noreferrer">itch.io</a> 

## Gameplay Video

{% include video id="hr7SA7ZzB7s" provider="youtube" %}

## Playable Web Build

<div class="itchio-widget-wrapper">
    <iframe frameborder="0" src="https://itch.io/embed/3703612?bg_color=141010&amp;fg_color=ffffff&amp;link_color=f21368&amp;border_color=5d5d5d" width="552" height="167"><a href="https://danimunteanu.itch.io/dreamcatcher">Dreamcatcher by DaniMunteanu</a></iframe>
</div>

## Implemented Mechanics

### Procedural generation

{% include figure popup=true image_path="/assets/images/dreamcatcher/generation-preview.gif" alt="Zoomed out preview of multiple floor generations" caption="Zoomed out preview of multiple floor generations" %}

{% include figure popup=true image_path="/assets/images/dreamcatcher/room-indexes.png" alt="Every room spot has its own index" caption="Every room spot has its own index" %}

{% include figure popup=true image_path="/assets/images/dreamcatcher/door-indexes.png" alt="Every door/wall spot has its own index" caption="Every door/wall spot has its own index" %}

```gdscript
# floor.gd

extends Node2D

# minimum number of rooms to generate per floor
const MINIMUM_ROOMS = 10

# stores the room scenes that can be instantiated, ordered by their assigned index
var rooms_res = []

# same as above, but for the "fog" layers displayed on every room besides the current one 
var room_fills_res = []

# same as above, but with the doors/walls, ordered by the door/wall index
var doors_res = []
var walls_res = []

# tracks the currently placed rooms' indexes
# contains an element for every room index
# if placed_rooms[x] is null, it means the room with index x was not added yet
# else, placed_rooms[x] stores the instance of the room that was added at that index
var placed_rooms = []

# same as above, but with doors/walls' indexes
var placed_doors_or_walls = []
var placed_doors_indexes = []

# every door connects two rooms: Room A <-> Room B
# evert element in this list is a pair (Room A index, Room B index)
# the pair's index is the door/wall index
var room_pairs = []

# markers are object storing the location coordinates where rooms/doors/walls will be instanced
var room_markers = []
var door_markers = []

# number of rooms that will be generated
var number_of_rooms = 0

# tracks the indexes of the rooms that could still allow neighbours to be generated
# (rooms who can still add doors to other rooms)
var extendable_rooms = []

var rng = RandomNumberGenerator.new()

func generate_rooms():
	# only stores the indexes of added rooms, except for the first one
	var potential_shop_spawn_rooms = []
	
	# randomly pick the number of rooms to be generated
	number_of_rooms = rng.randi_range(MINIMUM_ROOMS, MINIMUM_ROOMS+2)
	
	# room with index 0 is always the first one generated
	placed_rooms[0] = rooms_res[0].instantiate()
	placed_rooms[0].global_position = room_markers[0].global_position
	
	add_child(placed_rooms[0])
	extendable_rooms.append(0)
	
	for i in range (1,number_of_rooms):
		# next room to be generated should always be a neighbour of one of the previously added rooms
		var room_to_extend = extendable_rooms.pick_random()
		
		# pick a random, not yet added, neighbour
		var next_room = placed_rooms[room_to_extend].next_room()
		
		# instantiate the respective room scene
		placed_rooms[next_room] = rooms_res[next_room].instantiate()
		placed_rooms[next_room].global_position = room_markers[next_room].global_position
		
		# update its neighbor data, but also the data of all previously placed rooms next to it
		placed_rooms[next_room].update_neighbours(placed_rooms, extendable_rooms)
		add_child(placed_rooms[next_room])
		
		# every newly added room can be picked to have a shop NPC inside it
		potential_shop_spawn_rooms.append(next_room)
		
		# if the newly added room can still have neighbours, store it
		if placed_rooms[next_room].can_extend:
			extendable_rooms.append(next_room)
	
	# randomly pick which of the added rooms to host the shop NPC
	var shop_room_index = potential_shop_spawn_rooms.pick_random()
	placed_rooms[shop_room_index].is_shop_room = true
	Global.shop_room_picked.emit(shop_room_index)

func generate_doors_and_walls():
	# loop through every possible room pairing established in placed_rooms[]
	for j in 48:
		var new_door_or_wall
		
		# if the current pair's rooms have both been added, then a new door between them should also be instantiated 
		if (placed_rooms[room_pairs[j][0]] != null) and (placed_rooms[room_pairs[j][1]] != null):
			new_door_or_wall = doors_res[j].instantiate()
			new_door_or_wall.set_rooms(room_pairs[j][0],room_pairs[j][1])
			placed_doors_indexes.append(j)
		else:
			# if not, a wall should be instantiated
			new_door_or_wall = walls_res[j].instantiate()
			
		# regardless, the door/wall instance should be added and stored 
		new_door_or_wall.global_position = door_markers[j].global_position
		add_child(new_door_or_wall)
		placed_doors_or_walls[j] = new_door_or_wall
```

### Save file system

{% include figure popup=true image_path="/assets/images/dreamcatcher/save-system-preview.gif" alt="Game can save and load the player's position, stats and current room" caption="Game can save and load the player's position, stats and current room" %}

### Shop and upgrade system

{% include figure popup=true image_path="/assets/images/dreamcatcher/resources-preview.gif" alt="Killing enemies grants randomly picked resources" caption="Killing enemies grants randomly picked resources" %}

{% include figure popup=true image_path="/assets/images/dreamcatcher/shop-preview.gif" alt="Shop UI offers stat upgrades by purchasing tomes. HP can also be restored for a price" caption="Shop UI offers stat upgrades by purchasing tomes. HP can also be restored for a price" %}