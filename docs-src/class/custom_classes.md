
# Custom Classes

## Data Class

```gd
# ============================================
# DATA CLASS - No visual representation
# ============================================

class_name PlayerData extends RefCounted

var player_name: String
var level: int
var gold: int

func _init(name: String = "Hero", start_level: int = 1):
    player_name = name
    level = start_level
    gold = 0

func add_gold(amount: int) -> void:
    gold += amount
```

## Scene Class

```gd
# ============================================
# SCENE CLASS - Visual game object
# ============================================
class_name Player extends CharacterBody2D

@export var move_speed: float = 200.0

var player_data: PlayerData

func _ready():
    player_data = PlayerData.new("Chris", 1)

func _physics_process(delta):
    # Movement logic here
    pass
```


## Manager Class

```gd
# ============================================
# MANAGER CLASS - Global logic
# ============================================
class_name GameManager extends RefCounted
# Or if you need scene tree access:
# class_name GameManager extends Node

signal game_started
signal game_over

var current_level: int = 1

func start_game() -> void:
    game_started.emit()
```

## Resource Class

```gd
# ============================================
# RESOURCE CLASS - Reusable data templates
# ============================================
class_name WeaponData extends Resource

@export var weapon_name: String
@export var damage: int
@export var icon: Texture2D
```

__When to use what:__

- `RefCounted` - Pure data/logic
- `Node` - Needs scene tree access OR is an autoload that uses tree features
- `Node2D/Node3D/Control/etc` - Actual game objects
- `Resource` - Reusable data templates (create .tres files)
