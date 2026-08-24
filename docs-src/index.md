# Godot 4 - 2D Rapid Reference

## Core Node Types

| Node Type | Purpose | Common Use Cases |
|-----------|---------|------------------|
| `Node2D` | Base 2D node with transform | Parent containers, pivot points |
| `Sprite2D` | Display single image | Characters, items, static objects |
| `AnimatedSprite2D` | Play sprite animations | Animated characters, effects |
| `CharacterBody2D` | Physics body with move_and_slide | Player, NPCs, moving entities |
| `RigidBody2D` | Physics-simulated body | Physics objects, ragdolls |
| `StaticBody2D` | Non-moving physics body | Walls, platforms, obstacles |
| `Area2D` | Detect overlaps/triggers | Collectibles, triggers, hitboxes |
| `CollisionShape2D` | Define collision area | Used with physics bodies |
| `Camera2D` | 2D camera view | Follow player, screen shake |
| `CanvasLayer` | UI layer above game | HUD, menus, overlays |
| `Control` | Base UI node | UI containers and elements |
| `Timer` | Count down timer | Delays, cooldowns, events |
| `AudioStreamPlayer2D` | Positional sound | Footsteps, projectiles |
| `TileMap` | Grid-based tile system (deprecated in 4.3) | Use `TileMapLayer` instead |
| `TileMapLayer` | Single-layer tile map (new in 4.3) | Levels, terrain, backgrounds |
|| `Line2D` | Draw lines | Trails, paths, debug |
| `Polygon2D` | Draw filled shapes | Custom shapes, effects |

## Transform Properties (2D)
| Property | Type | Description |
|----------|------|-------------|
| `position` | Vector2 | World position (x, y) |
| `global_position` | Vector2 | Position in world space |
| `rotation` | float | Rotation in radians |
| `rotation_degrees` | float | Rotation in degrees |
| `scale` | Vector2 | Scale multiplier (x, y) |

## Visual Properties
| Property | Type | Description |
|----------|------|-------------|
| `visible` | bool | Show/hide node |
| `modulate` | Color | Tint color and alpha |
| `self_modulate` | Color | Tint without affecting children |
| `z_index` | int | Draw order (-4096 to 4096) |
| `z_as_relative` | bool | Z-index relative to parent |

## Physics Properties (CharacterBody2D)
| Property | Type | Description |
|----------|------|-------------|
| `velocity` | Vector2 | Current movement velocity |
| `floor_stop_on_slope` | bool | Stop sliding on slopes |
| `floor_max_angle` | float | Maximum walkable angle |

## Node Functions
| Function | Description |
|----------|-------------|
| `_ready()` | Called when node enters scene tree |
| `_process(delta)` | Called every frame |
| `_physics_process(delta)` | Called every physics frame (60 FPS) |
| `queue_free()` | Delete node at end of frame |
| `get_node("path")` or `$path` | Get reference to child node |
| `get_parent()` | Get parent node |
| `add_child(node)` | Add child node |
| `remove_child(node)` | Remove child node |
| `reparent(new_parent)` | Move to new parent |

## CharacterBody2D Functions
| Function | Description |
|----------|-------------|
| `move_and_slide()` | Move with collision detection |
| `is_on_floor()` | Check if on floor |
| `is_on_wall()` | Check if touching wall |
| `is_on_ceiling()` | Check if touching ceiling |
| `get_floor_normal()` | Get floor surface normal |

## Area2D Signals
| Signal | Description |
|--------|-------------|
| `body_entered(body)` | When a body enters area |
| `body_exited(body)` | When a body exits area |
| `area_entered(area)` | When another area enters |
| `area_exited(area)` | When another area exits |

## Input Functions
| Function | Description |
|----------|-------------|
| `Input.is_action_pressed("action")` | Check if action is held |
| `Input.is_action_just_pressed("action")` | Check if just pressed this frame |
| `Input.is_action_just_released("action")` | Check if just released |
| `Input.get_vector("left", "right", "up", "down")` | Get movement vector |
| `Input.get_axis("negative", "positive")` | Get 1D axis (-1 to 1) |

## Scene Tree Functions

| Function | Description |
|----------|-------------|
| `get_tree().paused = true` | Pause game |
| `get_tree().reload_current_scene()` | Restart current scene |
| `get_tree().change_scene_to_file("path")` | Load new scene |
| `get_tree().change_scene_to_packed(scene)` | Load preloaded scene |
| `get_tree().quit()` | Exit game |
| `get_tree().create_timer(seconds)` | Create one-shot timer |

## Resource Loading

| Function | Description |
|----------|-------------|
| `load("res://path")` | Load resource (blocks game) |
| `preload("res://path")` | Load at compile time |
| `ResourceLoader.load_threaded_request("path")` | Async load |

## Vector2 Math

| Function/Constant | Description |
|-------------------|-------------|
| `Vector2(x, y)` | Create vector |
| `Vector2.ZERO` | (0, 0) |
| `Vector2.ONE` | (1, 1) |
| `Vector2.UP` | (0, -1) |
| `Vector2.DOWN` | (0, 1) |
| `Vector2.LEFT` | (-1, 0) |
| `Vector2.RIGHT` | (1, 0) |
| `vector.length()` | Get magnitude |
| `vector.normalized()` | Get unit vector |
| `vector.distance_to(other)` | Distance between points |
| `vector.direction_to(other)` | Normalized direction |
| `vector.lerp(target, weight)` | Linear interpolation |
| `vector.move_toward(target, delta)` | Move toward target |

## Collision Layers & Masks

| Function | Description |
|----------|-------------|
| `set_collision_layer_value(layer, bool)` | Enable/disable layer |
| `set_collision_mask_value(layer, bool)` | Enable/disable mask |
| `get_collision_layer_value(layer)` | Check layer state |

**Concept:**
- **Layer** = "I am on these layers"
- **Mask** = "I collide with these layers"

## AnimatedSprite2D

| Function/Property | Description |
|-------------------|-------------|
| `play("animation_name")` | Play animation |
| `stop()` | Stop animation |
| `animation_finished` signal | When animation completes |
| `frame` | Current frame number |
| `speed_scale` | Playback speed multiplier |

## Timer Functions

| Function/Property | Description |
|-------------------|-------------|
| `start(seconds)` | Start/restart timer |
| `stop()` | Stop timer |
| `timeout` signal | Emitted when timer ends |
| `wait_time` | Duration in seconds |
| `one_shot` | Fire once or loop |
| `autostart` | Start on _ready() |

## Camera2D

| Property | Description |
|----------|-------------|
| `enabled` | Activate this camera |
| `zoom` | Camera zoom level (Vector2) |
| `offset` | Camera offset from position |
| `position_smoothing_enabled` | Smooth camera movement |
| `position_smoothing_speed` | Smoothing speed |
| `drag_horizontal_enabled` | Enable horizontal drag |
| `drag_vertical_enabled` | Enable vertical drag |

## Utility Functions

| Function | Description |
|----------|-------------|
| `lerp(from, to, weight)` | Linear interpolation |
| `clamp(value, min, max)` | Limit value to range |
| `move_toward(from, to, delta)` | Move toward value |
| `remap(value, istart, istop, ostart, ostop)` | Remap range |
| `abs(value)` | Absolute value |
| `sign(value)` | Returns -1, 0, or 1 |
| `deg_to_rad(degrees)` | Convert to radians |
| `rad_to_deg(radians)` | Convert to degrees |
| `randf()` | Random float 0.0-1.0 |
| `randi()` | Random integer |
| `randf_range(from, to)` | Random float in range |
| `randi_range(from, to)` | Random int in range |

## Common Constants

| Constant | Value | Usage |
|----------|-------|-------|
| `PI` | 3.14159... | Circle calculations |
| `TAU` | 6.28318... | Full rotation (2π) |
| `INF` | Infinity | Math comparisons |
| `NAN` | Not a Number | Error checking |

## Signal Connection

```php
# Code connection
$Node.signal_name.connect(_on_signal)

# Disconnect
$Node.signal_name.disconnect(_on_signal)

# Check if connected
$Node.signal_name.is_connected(_on_signal)
```

## Tween Quick Reference

| Function | Description |
|----------|-------------|
| `create_tween()` | Create new tween |
| `tween.tween_property(object, property, value, duration)` | Animate property |
| `tween.tween_interval(seconds)` | Add delay |
| `tween.tween_callback(callable)` | Call function |
| `tween.set_parallel(bool)` | Parallel mode |
| `tween.set_loops(count)` | Repeat animation |
| `tween.kill()` | Stop and destroy tween |

## Common @export Types

```php
@export var speed: float = 100.0
@export var health: int = 100
@export var player_name: String = "Hero"
@export var enabled: bool = true
@export var sprite: Texture2D
@export var sound: AudioStream
@export var next_scene: PackedScene
@export_range(0, 100) var volume: int = 50
@export_enum("Easy", "Medium", "Hard") var difficulty: String
@export_flags("Run", "Jump", "Attack") var abilities: int
```

## Delta Time Usage

```php
# Always multiply movement by delta for frame-independent speed
velocity.x = speed * delta        # pixels per second
rotation += angular_speed * delta  # radians per second
timer += delta                     # seconds
```

## Quick Code Snippets

### Basic Movement
```php
func _physics_process(delta):
    var direction = Input.get_vector("left", "right", "up", "down")
    velocity = direction * speed
    move_and_slide()
```

### Platformer Movement
```php
func _physics_process(delta):
    if not is_on_floor():
        velocity.y += gravity * delta
    
    var input = Input.get_axis("left", "right")
    velocity.x = input * speed
    
    if Input.is_action_just_pressed("jump") and is_on_floor():
        velocity.y = jump_velocity
    
    move_and_slide()
```

### Follow Mouse
```php
func _process(delta):
    global_position = get_global_mouse_position()
```

### Look At Target
```php
func _process(delta):
    look_at(target.global_position)
```

### Spawn Object
```php
func spawn_object():
    var instance = preload("res://object.tscn").instantiate()
    instance.position = spawn_position
    get_parent().add_child(instance)
```

## New & Changed in Godot 4.4–4.7

!!! tip "Handy 2D-focused highlights"
    - **TileMapLayer** (4.3+) — the new preferred way to build tile maps. `TileMap` still works but is deprecated.
    - **Typed Dictionaries** (4.4+) — use `var my_dict: Dictionary[int, String]`.
    - **Jolt 3D Physics** (4.4+, default in 4.6) — faster 3D physics; 2D projects are unchanged.
    - **Embedded Game Window & Interactive In-Game Editing** (4.4+) — test and tweak without leaving the editor.
    - **Scene Painter** (4.7+) — paint scenes directly in the 2D viewport.
    - **Control Offset Transforms** (4.7+) — extra per-control positional transforms for UI.
    - **Full release notes:** https://godotengine.org/releases/

### Typed Dictionaries Example (4.4+)

```php
# Map enemy IDs to names
var enemy_names: Dictionary[int, String] = {
    1: "Goblin",
    2: "Orc",
    3: "Dragon"
}

# Map positions to scores
var best_scores: Dictionary[Vector2, int] = {}
```

### TileMapLayer Quick Start (4.3+)

```php
# Each layer is its own node; group them under a Node2D for multi-layer maps
# Use a TileMapLayer node and assign the same TileSet to all layers
```
