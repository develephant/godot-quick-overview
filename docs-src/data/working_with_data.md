# Working with Data in Code

Godot 4’s GDScript has a few built-in container types. Most game data can be modeled with `Array`, `Dictionary`, and custom `RefCounted` classes. For special cases you can build `Set`, `Stack`, and `Queue` patterns with the built-ins.

## Arrays

```gd
# ============================================
# Array basics
# ============================================
var items: Array = ["sword", "shield", "potion"]
items.append("axe")
items.erase("shield")
var first = items[0]
```

### Typed Arrays

```gd
var scores: Array[int] = [10, 20, 30]
# scores.append("text")  # Compile error
```

### Array as Stack

```gd
var stack: Array[int] = []
stack.append(1)
stack.append(2)
var top = stack.pop_back()  # 2
```

## Dictionaries

```gd
# ============================================
# Dictionary basics
# ============================================
var player: Dictionary = {
    "name": "Hero",
    "gold": 100,
    "inventory": []
}

player["gold"] += 50
print(player.get("xp", 0))  # default if missing
```

### Typed Dictionaries

```gd
var stats: Dictionary[String, int] = {
    "str": 10,
    "dex": 14
}
```

## Globals

Use an Autoload (Project > Autoload) for global state.

```gd
# ============================================
# globals.gd — Autoload singleton
# ============================================
extends Node

var player_gold: int = 0
var unlocked_levels: Array[String] = []

func add_gold(amount: int):
    player_gold += amount
```

__Usage:__

```gd
Globals.add_gold(50)
print(Globals.player_gold)
```

## Data Classes

A `RefCounted` class is perfect for pure data. It auto-frees when no longer referenced.

```gd
# ============================================
# player_data.gd
# ============================================
class_name PlayerData extends RefCounted

var name: String
var gold: int
var inventory: Array[String]

func _init(_name: String, _gold: int = 0):
    name = _name
    gold = _gold
    inventory = []
```

__Usage:__

```gd
var data = PlayerData.new("Hero", 100)
data.inventory.append("sword")
```

## Other In-Memory Data Structures

### Set

Use a `Dictionary` with a dummy value for O(1) lookups.

```gd
var visited: Dictionary = {}
visited["level_1"] = true
if visited.has("level_1"):
    print("Already seen")
```

### Queue

Use an `Array` with `append` and `pop_front`.

```gd
var queue: Array[String] = []
queue.append("enemy_a")
queue.append("enemy_b")
var next = queue.pop_front()
```

### Priority Queue

Godot has no built-in priority queue. Use a sorted `Array` and `insert` where it belongs, or use a `RefCounted` wrapper over a min-heap `Array`.

```gd
var tasks: Array[Dictionary] = []
# Each task: { "name": String, "priority": int }

func add_task(name: String, priority: int):
    var task = {"name": name, "priority": priority}
    var idx = 0
    for i in range(tasks.size()):
        if tasks[i]["priority"] > priority:
            break
        idx = i + 1
    tasks.insert(idx, task)

func next_task() -> Dictionary:
    return tasks.pop_front()
```
