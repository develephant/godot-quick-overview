
# Input Events

## Input Map Settings

Navigate to __Project Settings > Input Map__ in the editor.

### Define Actions

- ___move_left___ -> `A`, `Left Arrow`
- ___move_right___ -> `D`, `Right Arrow`
- ___jump___ -> `Space`
- ___attack___ -> `Mouse Button Left`

## Polling Input

```gd
# ============================================
# POLLING INPUT (every frame)
# ============================================
extends CharacterBody2D

func _physics_process(delta):
    # Digital input (pressed/not pressed):
    var direction = Input.get_axis("move_left", "move_right")
    velocity.x = direction * move_speed
    
    # Jump (just pressed this frame):
    if Input.is_action_just_pressed("jump"):
        velocity.y = -jump_force
    
    # Attack (held down):
    if Input.is_action_pressed("attack"):
        charge_attack(delta)
    
    move_and_slide()
```

## Event Based Input

```gd
# ============================================
# EVENT-BASED INPUT
# ============================================
extends Node

func _unhandled_input(event):
    # Keyboard:
    if event is InputEventKey:
        if event.pressed and event.keycode == KEY_ESCAPE:
            toggle_pause()
    
    # Mouse button:
    if event is InputEventMouseButton:
        if event.button_index == MOUSE_BUTTON_LEFT and event.pressed:
            shoot(event.position)
    
    # Mouse motion:
    if event is InputEventMouseMotion:
        aim_at(event.position)
```

## Button Signals

```gd
# ============================================
# UI BUTTON SIGNALS
# ============================================
extends Control

@onready var start_button: Button = $StartButton

func _ready():
    start_button.pressed.connect(_on_start_pressed)
    
    # Or multi-param:
    start_button.pressed.connect(func(): start_game(1))

func _on_start_pressed():
    print("Game starting!")
    GameManager.change_scene("res://scenes/game.tscn")
```

## Sprite Click Detection

```gd
# ============================================
# CLICK DETECTION ON SPRITES
# ============================================
extends Sprite2D

signal clicked

func _ready():
    # Make it clickable (needs CollisionShape2D child):
    var area = Area2D.new()
    var collision = CollisionShape2D.new()
    var shape = RectangleShape2D.new()
    shape.size = texture.get_size()
    collision.shape = shape
    area.add_child(collision)
    add_child(area)
    
    area.input_event.connect(_on_input_event)

func _on_input_event(viewport, event, shape_idx):
    if event is InputEventMouseButton:
        if event.button_index == MOUSE_BUTTON_LEFT and event.pressed:
            clicked.emit()
            print("Sprite clicked!")
```
