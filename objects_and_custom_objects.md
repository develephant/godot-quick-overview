# Objects in Godot 4

## Built-in Object Types

```php
# ============================================
# OBJECT - Base class for everything
# ============================================
# Object is the root class that everything inherits from
# You rarely use Object directly

# ============================================
# REFCOUNTED - Memory-managed objects
# ============================================
# Automatically deleted when no longer referenced
# Use for data containers and logic classes

class_name InventorySystem extends RefCounted

var items: Array[Item] = []
var max_slots: int = 20

func add_item(item: Item) -> bool:
    if items.size() < max_slots:
        items.append(item)
        return true
    return false

# Usage:
var inventory = InventorySystem.new()  # Created
inventory.add_item(sword)
inventory = null  # Automatically freed when no references exist

# ============================================
# NODE - Scene tree objects
# ============================================
# Part of the scene tree, has lifecycle methods
# Can have parent/children relationships

class_name GameController extends Node

func _ready():
    print("Node entered scene tree")

func _process(delta):
    # Called every frame
    pass

# ============================================
# RESOURCE - Serializable data
# ============================================
# Can be saved to .tres files
# Shared between instances by default

class_name ItemData extends Resource

@export var item_name: String
@export var icon: Texture2D
@export var value: int

# Save as item_sword.tres and reuse everywhere
```

## Object vs RefCounted vs Node

| Type | Memory | Scene Tree | Lifecycle | Best For |
|------|--------|------------|-----------|----------|
| `Object` | Manual | No | Manual | Rarely used directly |
| `RefCounted` | Auto | No | Reference counting | Data, logic, utilities |
| `Node` | Manual* | Yes | _ready, _process, etc | Game objects, scenes |
| `Resource` | Auto | No | Reference counting | Reusable data templates |

*Nodes are freed when removed from tree or parent is freed

## Creating Custom Objects

### Custom Data Object (RefCounted)

```php
# ============================================
# USE CASE: Pure data container
# ============================================
class_name QuestData extends RefCounted

var quest_id: String
var title: String
var description: String
var objectives: Array[String] = []
var is_completed: bool = false
var reward_gold: int = 0

func _init(id: String, quest_title: String):
    quest_id = id
    title = quest_title

func complete() -> void:
    is_completed = true
    print("Quest completed: ", title)

func add_objective(objective: String) -> void:
    objectives.append(objective)

# ADVANTAGE: Lightweight, no scene tree overhead
# USAGE: When you need structured data without visuals

# Example:
var quest = QuestData.new("quest_001", "Save the Village")
quest.add_objective("Defeat 10 goblins")
quest.add_objective("Return to elder")
quest.reward_gold = 100
```

### Custom Logic Object (RefCounted)

```php
# ============================================
# USE CASE: Reusable logic/algorithms
# ============================================
class_name PathfindingHelper extends RefCounted

func find_path(start: Vector2, end: Vector2, obstacles: Array) -> Array[Vector2]:
    # Pathfinding algorithm here
    var path: Array[Vector2] = []
    # ... calculate path
    return path

func get_nearest_point(position: Vector2, points: Array[Vector2]) -> Vector2:
    var nearest = points[0]
    var min_distance = position.distance_to(nearest)
    
    for point in points:
        var distance = position.distance_to(point)
        if distance < min_distance:
            min_distance = distance
            nearest = point
    
    return nearest

# ADVANTAGE: No scene tree dependency, easy to test
# USAGE: Utilities, algorithms, math helpers

# Example:
var pathfinder = PathfindingHelper.new()
var path = pathfinder.find_path(player_pos, enemy_pos, walls)
```

### Custom State Machine (RefCounted)

```php
# ============================================
# USE CASE: State management without Node overhead
# ============================================
class_name StateMachine extends RefCounted

signal state_changed(old_state: String, new_state: String)

var current_state: String = ""
var states: Dictionary = {}

func add_state(state_name: String, enter_func: Callable, exit_func: Callable) -> void:
    states[state_name] = {
        "enter": enter_func,
        "exit": exit_func
    }

func change_state(new_state: String) -> void:
    if not states.has(new_state):
        push_error("State doesn't exist: " + new_state)
        return
    
    # Exit current state
    if current_state != "" and states[current_state].has("exit"):
        states[current_state]["exit"].call()
    
    var old_state = current_state
    current_state = new_state
    
    # Enter new state
    if states[new_state].has("enter"):
        states[new_state]["enter"].call()
    
    state_changed.emit(old_state, new_state)

# ADVANTAGE: Lightweight, reusable across different nodes
# USAGE: AI states, animation states, game states

# Example:
class_name Enemy extends CharacterBody2D

var state_machine: StateMachine

func _ready():
    state_machine = StateMachine.new()
    state_machine.add_state("idle", _enter_idle, _exit_idle)
    state_machine.add_state("chase", _enter_chase, _exit_chase)
    state_machine.add_state("attack", _enter_attack, _exit_attack)
    state_machine.change_state("idle")

func _enter_idle(): print("Now idle")
func _exit_idle(): print("Leaving idle")
func _enter_chase(): print("Now chasing")
func _exit_chase(): print("Stop chasing")
```

### Custom Resource Object

```php
# ============================================
# USE CASE: Reusable data templates
# ============================================
class_name EnemyStats extends Resource

@export_group("Basic Info")
@export var enemy_name: String = "Goblin"
@export var sprite: Texture2D

@export_group("Stats")
@export var max_health: int = 50
@export var move_speed: float = 100.0
@export var damage: int = 10

@export_group("Drops")
@export var gold_drop: int = 25
@export var experience: int = 50

# ADVANTAGE: Create .tres files, share between enemies
# USAGE: When you need multiple instances with different values

# Create goblin_stats.tres, orc_stats.tres, etc.
# Then in enemy script:

class_name Enemy extends CharacterBody2D

@export var stats: EnemyStats

var current_health: int

func _ready():
    if stats:
        current_health = stats.max_health
        $Sprite2D.texture = stats.sprite
```

## Use Cases for Custom Objects

### 1. Inventory System

```php
# ============================================
# INVENTORY SYSTEM - RefCounted
# ============================================
class_name Inventory extends RefCounted

signal item_added(item: Item)
signal item_removed(item: Item)

var items: Array[Item] = []
var max_capacity: int = 20

func add_item(item: Item) -> bool:
    if items.size() >= max_capacity:
        return false
    
    items.append(item)
    item_added.emit(item)
    return true

func remove_item(item: Item) -> bool:
    var index = items.find(item)
    if index != -1:
        items.remove_at(index)
        item_removed.emit(item)
        return true
    return false

func has_item(item_name: String) -> bool:
    for item in items:
        if item.item_name == item_name:
            return true
    return false

func get_total_weight() -> float:
    var weight = 0.0
    for item in items:
        weight += item.weight
    return weight

# WHY CUSTOM: Encapsulates inventory logic, easy to test
```

### 2. Dialogue System

```php
# ============================================
# DIALOGUE DATA - Resource
# ============================================
class_name DialogueData extends Resource

@export var lines: Array[String] = []
@export var speaker_name: String = ""
@export var portrait: Texture2D

# ============================================
# DIALOGUE MANAGER - RefCounted
# ============================================
class_name DialogueManager extends RefCounted

signal dialogue_started(data: DialogueData)
signal line_changed(line: String, index: int)
signal dialogue_ended

var current_dialogue: DialogueData
var current_line_index: int = 0

func start_dialogue(data: DialogueData) -> void:
    current_dialogue = data
    current_line_index = 0
    dialogue_started.emit(data)
    line_changed.emit(data.lines[0], 0)

func next_line() -> bool:
    current_line_index += 1
    if current_line_index >= current_dialogue.lines.size():
        dialogue_ended.emit()
        return false
    
    line_changed.emit(current_dialogue.lines[current_line_index], current_line_index)
    return true

# WHY CUSTOM: Separates dialogue data from UI logic
```

### 3. Save/Load System

```php
# ============================================
# SAVE DATA - RefCounted
# ============================================
class_name SaveData extends RefCounted

var player_position: Vector2
var player_health: int
var player_gold: int
var inventory_items: Array[String] = []
var completed_quests: Array[String] = []
var current_level: String

func to_dict() -> Dictionary:
    return {
        "player_position": {"x": player_position.x, "y": player_position.y},
        "player_health": player_health,
        "player_gold": player_gold,
        "inventory_items": inventory_items,
        "completed_quests": completed_quests,
        "current_level": current_level
    }

func from_dict(data: Dictionary) -> void:
    player_position = Vector2(data.player_position.x, data.player_position.y)
    player_health = data.player_health
    player_gold = data.player_gold
    inventory_items = data.inventory_items
    completed_quests = data.completed_quests
    current_level = data.current_level

# ============================================
# SAVE MANAGER - RefCounted
# ============================================
class_name SaveManager extends RefCounted

const SAVE_PATH = "user://savegame.json"

func save_game(save_data: SaveData) -> void:
    var file = FileAccess.open(SAVE_PATH, FileAccess.WRITE)
    var json = JSON.stringify(save_data.to_dict())
    file.store_string(json)
    file.close()

func load_game() -> SaveData:
    if not FileAccess.file_exists(SAVE_PATH):
        return null
    
    var file = FileAccess.open(SAVE_PATH, FileAccess.READ)
    var json = file.get_as_text()
    file.close()
    
    var data = JSON.parse_string(json)
    var save_data = SaveData.new()
    save_data.from_dict(data)
    return save_data

# WHY CUSTOM: Centralized save logic, easy to modify format
```

### 4. Damage Calculator

```php
# ============================================
# DAMAGE CALCULATOR - RefCounted
# ============================================
class_name DamageCalculator extends RefCounted

enum DamageType { PHYSICAL, FIRE, ICE, POISON }

func calculate_damage(base_damage: int, attacker_stats: Dictionary, defender_stats: Dictionary) -> int:
    var damage = base_damage
    
    # Apply attack modifier
    if attacker_stats.has("attack"):
        damage += attacker_stats.attack * 0.5
    
    # Apply defense reduction
    if defender_stats.has("defense"):
        damage -= defender_stats.defense * 0.3
    
    # Random variance
    damage *= randf_range(0.9, 1.1)
    
    return max(1, int(damage))

func calculate_critical(damage: int, crit_chance: float, crit_multiplier: float) -> int:
    if randf() < crit_chance:
        return int(damage * crit_multiplier)
    return damage

func apply_resistance(damage: int, damage_type: DamageType, resistances: Dictionary) -> int:
    if resistances.has(damage_type):
        var resistance = resistances[damage_type]
        damage *= (1.0 - resistance)
    return int(damage)

# WHY CUSTOM: Complex calculations isolated, easy to balance/test
```

## Advantages of Custom Objects

### 1. **Separation of Concerns**
```php
# Without custom objects - everything in one script
extends CharacterBody2D
# 500+ lines of movement, inventory, combat, quests, etc.

# With custom objects - organized
extends CharacterBody2D
var inventory: Inventory
var quest_log: QuestLog
var combat_stats: CombatStats
# Each system is separate and manageable
```

### 2. **Reusability**
```php
# Create once, use everywhere
var player_inventory = Inventory.new(20)  # 20 slots
var chest_inventory = Inventory.new(10)   # 10 slots
var shop_inventory = Inventory.new(50)    # 50 slots
```

### 3. **Testability**
```php
# Easy to test without running entire game
var inventory = Inventory.new()
assert(inventory.add_item(sword) == true)
assert(inventory.items.size() == 1)
```

### 4. **Memory Efficiency**
```php
# Resource shared between instances
@export var enemy_stats: EnemyStats  # Points to same .tres file
# All goblins share goblin_stats.tres - one copy in memory
```

### 5. **Type Safety**
```php
# Without custom object
var player_data = {
    "name": "Hero",
    "health": 100,
    "glod": 50  # Typo! Hard to find
}

# With custom object
var player_data = PlayerData.new()
player_data.glod = 50  # Compile error - catches typo
```

## When to Create Custom Objects

| Create Custom Object When... | Example |
|------------------------------|---------|
| Data needs structure | PlayerData, QuestData, SaveData |
| Logic is reusable | PathfindingHelper, DamageCalculator |
| System is complex | Inventory, DialogueManager |
| Need multiple variants | EnemyStats (goblin.tres, orc.tres) |
| Want to serialize data | Save system, level data |
| Reduce Node overhead | State machine, timers, counters |

## When NOT to Create Custom Objects

| Use Built-in When... | Use Node When... |
|----------------------|------------------|
| Simple data (use Dictionary) | Needs visual representation |
| One-time use (inline code) | Needs scene tree access |
| Godot has it (Timer, etc.) | Needs signals to UI |

__Best Practice:__
- Start simple with built-in types
- Extract to custom objects when code gets complex
- Use RefCounted for pure logic/data
- Use Resource for reusable templates
- Use Node only when scene tree is needed
