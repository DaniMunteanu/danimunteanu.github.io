---
permalink: /projects/skate-wizard-supreme
layout: single
author_profile: true
author: Daniel Munteanu
toc: true
toc_sticky: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

# Skate Wizard Supreme

## Short Description

This **2D survivors-like Godot** game was developed with the help of 4 friends in the span of 1 week, as an entry for  
<a href="https://itch.io/jam/theveryseriousjuniperdevgamejam" target="_blank" rel="noopener noreferrer">The Very Serious Juniper Dev Game Jam 2026</a>.

The jam's theme was **Spin to Win** and had an extra voting criteria for (Un)_Seriousness_. This is why we decided to make a game where you play as a wizard, who periodically attempts to execute ice-skating spinning techniques. At the same time, he throws an endless amount of spinning shurikens to defend against hostile Hockey Jocks, Polar Bears and Sharknados.

Every successful spinning move grants EXP points. Every level-up gives the player 3 upgrade options to choose from. However, 3 failed spins will make the wizard vomit rainbows on the ground, rendering him unable to move or throw shurikens for a short duration.

My most notable contributions to this project include:
- The spinning Quick Time Event
- The shuriken throwing system 

This project can also be found on 
<i class="fa-brands fa-github"></i>
<a href="https://github.com/DaniMunteanu/juniper-jam" target="_blank" rel="noopener noreferrer">Github</a> 
and 
<i class="fa-brands fa-itch-io"></i>
<a href="https://devwizards.itch.io/skate-wizard-supreme" target="_blank" rel="noopener noreferrer">itch.io</a> 

## Gameplay Video

{% include video id="zmwrUNrhURY" provider="youtube" %}

## Playable Web Build

<div class="itchio-widget-wrapper">
    <iframe frameborder="0" src="https://itch.io/embed/4718198?bg_color=141010&amp;fg_color=ffffff&amp;link_color=f21368&amp;border_color=5d5d5d" width="552" height="167"><a href="https://devwizards.itch.io/skate-wizard-supreme">Skate Wizard Supreme by Dev Wizards, DaniMunteanu, Willo_Flow, Lluke18, Natalia Rosu</a></iframe>
</div>

## Implemented Mechanics

### The spinning Quick Time Event

{% include figure popup=true image_path="/assets/images/skate-wizard-supreme/qte-success-preview.gif" alt="Successfully spinning the cursor grants EXP points" caption="Successfully spinning the cursor grants EXP points" %}

{% include figure popup=true image_path="/assets/images/skate-wizard-supreme/qte-fail-preview.gif" alt="Failing the quick time event temporarily stuns the player" caption="Failing the quick time event temporarily stuns the player" %}

```gdscript
# spin_minigame.gd

extends Control

var required_orientation: int = 1
var required_key: Key = KEY_E

# time elapsed since the quick-time-event started
var elapsed := 0.0
# currently accumulated rotation
var total_rotation := 0.0
var previous_angle := 0.0
var center_global_position: Vector2 = Vector2.ZERO

# When minigame starts
func _ready():
	generate_random()
	
	total_required_rotation = base_required_rotation + level * required_rotation_multiplier
	
	# The circular arrow sprite should be flipped horizontally if Counter-clockwise rotation is required
	center.flip_h = required_orientation < 0
	
	indication_label.text = "Hold " + OS.get_keycode_string(required_key) + "
	 + 
	spin the cursor"
	
	center_global_position = center.global_position + center.pivot_offset
	
	previous_angle = (get_global_mouse_position() - center_global_position).angle()

# On each minigame, the player needs to hold a key while rotating the cursor with a certain orientation
# The key and orientation are randomly picked
func generate_random():
	required_key = keys_list.pick_random()
	required_orientation = [-1,1].pick_random() #-1 = Counter-Clockwise, 1 = Clockwise

# Called every frame. 'delta' is the elapsed time since the previous frame.
func _process(delta):
	elapsed += delta
	timer_bar.value = duration - elapsed

	# The circular arrow's center point and Cursor's position form a line
	# Get the angle between that line and the X-axis, in radians
	var angle = (get_global_mouse_position() - center_global_position).angle()

	# Get the angle difference since last frame, and wrap it between -PI and PI
	var delta_angle = wrapf(angle - previous_angle, -PI, PI)
	
	#if required_orientation = -1, delta_angle needs to be negative (Counter-clockwise)
	#if required_orientation = 1, delta_angle needs to be positive (Clockwise)
	if Input.is_key_pressed(required_key) and required_orientation * delta_angle > 0.0 and abs(delta_angle) > 0.1:
		total_rotation += abs(delta_angle)
		required_score_bar.value = total_rotation
		bonus_score_bar.value = total_rotation - total_required_rotation
		center.rotation += delta_angle

	previous_angle = angle

	if elapsed >= duration:
		if total_rotation >= total_required_rotation:
			spinner_won()
		else:
			spinner_failed()
		minigame_over.emit()
		queue_free()
```

```gdscript
# This is player_canvas.gd, a script with a periodic timer which triggers a quick time event

#start minigame
func _on_timer_timeout() -> void:
	player_camera.zoom_in()
	time_scaler.time_scale = 0.1
	
	var new_minigame = SPIN_MINIGAME.instantiate()
	add_child(new_minigame)
	new_minigame.level = level
	
	minigame_timer.stop()
```

{% include figure popup=true image_path="/assets/images/skate-wizard-supreme/time-scaler.png" alt="Resource object used to store current time acceleration" caption="Resource object used to store current time acceleration" %}

{% include figure popup=true image_path="/assets/images/skate-wizard-supreme/enemy-time-scaler.png" alt="The current acceleration value is used to determine enemy movement speed" caption="The current acceleration value is used to determine enemy movement speed" %}

```gdscript
# polar_bear.gd, script of the Polar bear enemy

func _physics_process(delta: float) -> void:
	var direction = global_position.direction_to(player.global_position)
	
	# When a quick time event starts, time_scaler.time_scale = 0.1 for slow-motion effect
	velocity = direction * stats.curr_speed * time_scaler.time_scale
	update_animation()
			
	# Moves the enemy based on its velocity
	move_and_slide()
```

### The shuriken throwing system

{% include figure popup=true image_path="/assets/images/skate-wizard-supreme/shurikens-preview.gif" alt="Shurikens are thrown in a pattern" caption="Shurikens are thrown in a pattern" %}

This implementation is a 2D Godot adaptation of the 
<a href="/projects/bullet-hell-generator" target="_blank" rel="noopener noreferrer">Bullet-Hell Generator project</a>.

```gdscript
extends Node2D
class_name ShurikenSpawner

@export var combat_stats: CombatStats

# Repeating timer setting off the shuriken throwing
@onready var timer: Timer = $Timer

# Shurikens shooting points are equally distanced on a circle 
# Horizontal angle deviates that position clockwise by an amount
@export var horizontal_angle: float = 10

# This further deviates that position on every shuriken ring
@export var horizontal_angle_step: float = 10

# Called when the player is spawned
func _ready() -> void:
	timer.wait_time = combat_stats.fire_rate
	timer.start()

func _on_timer_timeout() -> void:
	timer.wait_time = combat_stats.fire_rate
	if !is_puking and !is_in_minigame:
		spawn_shurikens()

func spawn_shurikens():
	var start_point = global_position
	# Shurikens are thrown from equally distant sides, depending how many are thrown at once
	var horizontal_angle_spacing = 360.0 / combat_stats.current_number_of_shurikens;
	
	for i in range(0, combat_stats.current_number_of_shurikens):
		var shuriken_dir_x_position = start_point.x + cos(((horizontal_angle + i * horizontal_angle_spacing) * PI) / 180.0) * combat_stats.range
		var shuriken_dir_y_position = start_point.y + sin(((horizontal_angle + i * horizontal_angle_spacing) * PI) / 180.0) * combat_stats.range
		
		var new_position_vector = Vector2(shuriken_dir_x_position, shuriken_dir_y_position)
		var shuriken_direction = (new_position_vector - start_point).normalized()
		
		# Picks between a normal shuriken and a critical one (x2 damage) based on crit chance stat
		var new_shuriken = pick_random_shuriken()
		new_shuriken.global_position = start_point
		new_shuriken.start_position = global_position
		
		new_shuriken.move_direction = shuriken_direction

		new_shuriken.horizontal_angle = horizontal_angle
		new_shuriken.horizontal_angle_spacing = i * horizontal_angle_spacing
		
		get_parent().get_parent().add_child(new_shuriken)
	
		if horizontal_angle >= 360.0:
			horizontal_angle -= 360.0
			
	horizontal_angle += horizontal_angle_step
```