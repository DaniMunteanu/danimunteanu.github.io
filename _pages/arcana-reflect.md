---
permalink: /projects/arcana-reflect
layout: single
author_profile: true
author: Daniel Munteanu
toc: true
toc_sticky: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

# Arcana Reflect

## Short Description

This game started as an entry for the 
<a href="https://rgda.ro/game-jam-universitar-2026/" target="_blank" rel="noopener noreferrer">2026 RGDA University Game Jam</a>.
I worked on this project along with other 5 students with which I've never collaborated before.
Despite this, development went smoothly and we even agreed to expand the game after the game jam ended.

The result is a **2D (and 3D!) puzzle game** built using the **Godot** engine.
The features I implemented include:
- Drag & Drop 2D puzzles
- Inventory system
- Puzzle progression tracking using a notebook UI

This project can also be found on 
<a href="https://github.com/DaniMunteanu/Game-Jam-Game" target="_blank" rel="noopener noreferrer">Github</a> 
and 
<a href="https://biesbi.itch.io/arcana-reflect" target="_blank" rel="noopener noreferrer">itch.io</a> 

## Gameplay Video

{% include video id="JveU_W238AE" provider="youtube" %}

## Download

<div class="itchio-widget-wrapper">
    <iframe frameborder="0" src="https://itch.io/embed/4511997?bg_color=141010&amp;fg_color=ffffff&amp;link_color=f21368&amp;border_color=5d5d5d" width="552" height="167"><a href="https://biesbi.itch.io/arcana-reflect">Arcana Reflect by BiesBi, DaniMunteanu, Natalia Rosu, Lluke18, Willo_Flow, RaluAndreee, Dev Wizards</a></iframe>
</div>

## Implemented Features

### Drag & Drop 2D Puzzles

As the name suggests, this mechanic allows the player to drag around objects and snap them to certain areas.

{% include figure popup=true image_path="/assets/images/arcana-reflect/chess-preview.gif" alt="Chess puzzle preview" caption="Chess puzzle preview" %}

{% include figure popup=true image_path="/assets/images/arcana-reflect/paintings-preview.gif" alt="Paintings puzzle preview" caption="Paintings puzzle preview" %}

In this section I will explain how I implemented the drag-and-drop system used in the 2 GIFs above. I will use the paintings puzzle as an example.

{% include figure popup=true image_path="/assets/images/arcana-reflect/snap-markers.png" alt="4 paintings must be placed at 4 position markers" caption="4 paintings must be placed at 4 position markers" %}

Every painting is an object of class **DraggablePuzzleObject**. It inherits the predefined **TextureButton** class because I want to detect when such an object is _pressed_ or _released_.

```gdscript
class_name DraggablePuzzleObject

extends TextureButton

var min_position: Vector2 = Vector2(0, 0)
var max_position: Vector2 = Vector2(1920, 1080)

# Called every frame. 'delta' is the elapsed time since the previous frame.
func _process(delta: float) -> void:
	if is_dragging and draggable:
		#Limit the new global position of the painting so it can't be moved to an offscreen position
		global_position.x = clamp(get_global_mouse_position().x - offset.x, min_position.x, max_position.x)
		global_position.y = clamp(get_global_mouse_position().y - offset.y, min_position.y, max_position.y)

#When dragging begins
func _on_button_down() -> void:
	is_dragging = true
	offset = get_global_mouse_position() - global_position

#When dragging ends
func _on_button_up() -> void:
	is_dragging = false
	if draggable:
		#Emit a signal so puzzles can check if the painting with a certain index could snap to a marker
		try_snapping.emit(index)
```

Below is the logic used for the painting puzzle scene. The markers and paintings are stored inside separate arrays. I also used an array to track the current state of the puzzle (_which_ painting is placed _where_).

```gdscript
var markers: Array[Marker2D] = []
var paintings: Array[DraggablePuzzleObject] = []
#Index of this array is the index of the painting, and the value stored is the index of the marker
#If the painting is not snapped to any marker, the value is -1
var paintings_snapped: Array[int] = []

#Store all paintings in an array
func init_paintings_array():
	for child in paintings_parent.get_children():
		if child is DraggablePuzzleObject:
			#Connecting every painting to a method which will be called everytime a drag ends
			child.try_snapping.connect(on_try_snapping)
			paintings.append(child)

func on_try_snapping(painting_index: int):
	var closest_marker_index = -1
	var closest_distance = snap_max_distance
	var painting_center = paintings[painting_index].global_position + paintings[painting_index].size / 2
	
	#Checks what is the closest marker within the maximum snap distance
	for marker_index in range(markers.size()):
		var dist = painting_center.distance_to(markers[marker_index].global_position)
		if dist < closest_distance:
			closest_distance = dist
			closest_marker_index = marker_index
	
	if closest_marker_index != -1:
		#If the found marker already has a different painting snapped, don't snap the current painting anywhere
		if paintings_snapped.has(closest_marker_index) and paintings_snapped[painting_index] != closest_marker_index:
			paintings_snapped[painting_index] = -1
			return
		#Otherwise, update the array and move the painting so the center ends up in the marker's position
		paintings_snapped[painting_index] = closest_marker_index
		paintings[painting_index].global_position = markers[closest_marker_index].global_position - paintings[painting_index].size / 2
		
		#Each painting has the same index as it's correct marker index, so
		#Painting 0 should be placed at Marker 0, Painting 1 should be placed at Marker 1...
		if closest_marker_index == painting_index:
			TextManager.show_text(painting_descriptions[painting_index])
			check_if_solved()
	else:
		#If no marker has been found nearby the drop location, don't do anything
		paintings_snapped[painting_index] = -1

func check_if_solved():
	#Checks if all paintings have matching index values with the markers they are snapped to
	for painting_index in range(paintings_snapped.size()):
		if paintings_snapped[painting_index] != painting_index:
			return
	end_puzzle()
```

### Inventory System

The player uses an inventory bar UI component to see all possessed items. They can also pick which item to use by scrolling up or down with their mouse.

{% include figure popup=true image_path="/assets/images/arcana-reflect/inventory-preview.gif" alt="Adding and removing items to the inventory" caption="Adding and removing items to the inventory" %}

{% include figure popup=true image_path="/assets/images/arcana-reflect/inventory-ui.png" alt="Inventory bar inside the editor" caption="Inventory bar inside the editor" %}

Below is the logic for selecting the item based on mouse scroll.

```gdscript
# Called every frame. 'delta' is the elapsed time since the previous frame.
func _process(delta: float) -> void:
	detect_scrolling()

func detect_scrolling():
	if Input.is_action_just_released("scroll_up"):
		if InventoryManager.selected_item_index > 0:
			slots[InventoryManager.selected_item_index].is_selected = false
			InventoryManager.selected_item_index -= 1
			update_selection()
	elif Input.is_action_just_released("scroll_down"):
		if InventoryManager.selected_item_index < (InventoryManager.MAX_SIZE - 1):
			slots[InventoryManager.selected_item_index].is_selected = false
			InventoryManager.selected_item_index += 1
			update_selection()
```

This part handles the inventory bar updates.

```gdscript
# Called on the first process frame
func _ready() -> void:
	init_slots_array()
	call_deferred("update_bar")
	#Everytime the inventory is modified, the "update_bar" method will be called
	InventoryManager.inventory_modified.connect(update_bar)

func update_bar():
	var slot_index: int = 0
	for item in InventoryManager.obtained_items:
		slots[slot_index].item = item
		slots[slot_index].update_ui()
		slot_index += 1
	
	#If an item was removed, the last slot needs to be emptied
	if slot_index < InventoryManager.MAX_SIZE:
		slots[slot_index].item = null
		slots[slot_index].update_ui()
	
	call_deferred("update_selection")
```

The items are stored inside **Resource Files**, the Godot equivalent for Unity **Scriptable Objects**.

{% include figure popup=true image_path="/assets/images/arcana-reflect/item-resource.png" alt="Item data in the editor" caption="
Item data in the editor" %}

In the previous code snippets there have been many mentions of **InventoryManager**. That is a global script file used to change the inventory state from any script. It also stores _const_ variables for every item.

```gdscript
#ITEMS
const MOONSIGN: ItemData = preload("res://Scripts/Items/MoonSign.tres")
const PENCIL: ItemData = preload("res://Scripts/Items/Pencil.tres")
const BLUE_POT: ItemData = preload("res://Scripts/Items/BluePot.tres")
#other items go here...

const MAX_SIZE = 8

var obtained_items: Array[ItemData] = []
var selected_item: ItemData
var selected_item_index: int = 0

signal inventory_modified

func send_inventory_modified_signal():
	inventory_modified.emit()

func remove_item(item: ItemData):
	obtained_items.erase(item)
	call_deferred("send_inventory_modified_signal")
	
func add_item(item: ItemData):
	if obtained_items.size() < MAX_SIZE:
		obtained_items.append(item)
		call_deferred("send_inventory_modified_signal")
```

### Puzzle Progression Tracking

{% include figure popup=true image_path="/assets/images/arcana-reflect/notebook-preview.gif" alt="Visual indicator of a puzzle completion" caption="Visual indicator of a puzzle completion" %}

Completion data is stored inside an array, storing a _bool_ value for each puzzle. There are also variables used to determine whether significant changes have occurred to any of the scenes. For instance, if an item was already picked up, it should never spawn again.
The code below belongs to another global script, **PuzzleManager**.

```gdscript
enum puzzles{
	MAGICIAN,
	HIEROPHANT,
	STAR,
	DEATH,
	TEMPERANCE,
	MOON,
	SUN,
	EMPEROR,
	JUDGEMENT,
	WORLD,
	FOOL,
}

#Flags used to preserve the game state when switching between scenes
var moon_symbol_picked_up: bool = false
var pencil_picked_up: bool = false
var calendar_solved: bool = false
var chest_opened : bool = false
#Other such variables...

const number_of_puzzles: int = 11
var complete_puzzles: Array[bool] = []
var completed_puzzles : int = 0

signal puzzle_finished(puzzle_index: int)

func finish_puzzle(puzzle_index: int):
	complete_puzzles[puzzle_index] = true
	puzzle_finished.emit(puzzle_index)
```

Whenever a puzzle is completed, the *puzzle_finished* signal is received by an UI component responsible for playing a fade in-out sequence for the puzzle's corresponding card.

```gdscript
var big_cards: Array[Card] = []

func _ready() -> void:
	init_big_cards()
	PuzzleManager.puzzle_finished.connect(on_puzzle_finished)

func on_puzzle_finished(puzzle_index: int):
	big_cards[puzzle_index].play_fade_sequence()
```

{% include figure popup=true image_path="/assets/images/arcana-reflect/notebook-pages.png" alt="Choosing the cards tracked by a notebook page" caption="Choosing the cards tracked by a notebook page" %}

```gdscript
@onready var card_left: TextureRect = $CardLeft
@onready var card_right: TextureRect = $CardRight

func update_cards():
	card_left.visible = PuzzleManager.complete_puzzles[left_card_index]
	card_right.visible = PuzzleManager.complete_puzzles[right_card_index]
```