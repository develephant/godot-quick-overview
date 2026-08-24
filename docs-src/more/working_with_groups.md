# Working with Groups

Groups in Godot are a powerful organizational system that allows you to tag nodes with string identifiers and then operate on collections of nodes efficiently. This is particularly useful for managing enemies, collectibles, UI elements, or any set of nodes that share common behavior.

## Adding Nodes to Groups

### In the Editor
1. Select a node in the Scene dock
2. Go to the Groups tab (next to Inspector)
3. Type a group name and click "Add"

### In Code
```gdscript
# Add to group during _ready()
func _ready():
    add_to_group("enemies")
    add_to_group("ai_controlled")

# Add another node to a group
some_node.add_to_group("collectibles")
```

## Retrieving Nodes from Groups

### Get All Nodes in a Group
```gdscript
# Get all enemy nodes
var enemies = get_tree().get_nodes_in_group("enemies")
for enemy in enemies:
    enemy.take_damage(10)

# Get first node in group (useful for singletons)
var player = get_tree().get_first_node_in_group("player")
if player:
    var player_position = player.global_position
```

### Check if Groups Exist
```gdscript
# Check if group has any nodes
if get_tree().has_group("enemies"):
    print("Enemies are present in the scene")

# Get group count
var enemy_count = get_tree().get_nodes_in_group("enemies").size()
print("There are ", enemy_count, " enemies")
```

## Group Operations

### Call Methods on Group Members
```gdscript
# Call a method on all nodes in a group
get_tree().call_group("enemies", "take_damage", 25)
get_tree().call_group("ui_elements", "hide")

# Call with flags for performance
get_tree().call_group_flags(
    SceneTree.GROUP_CALL_DEFERRED, 
    "enemies", 
    "queue_free"
)
```

### Set Properties on Group Members
```gdscript
# Set property on all group members
get_tree().set_group("enemies", "speed", 0)  # Freeze all enemies
get_tree().set_group("lights", "visible", false)  # Turn off all lights
```

## Removing from Groups

```gdscript
# Remove node from specific group
remove_from_group("enemies")

# Remove during cleanup
func _exit_tree():
    if is_in_group("persistent"):
        remove_from_group("persistent")
```

## Practical Examples

### Enemy Management
```gdscript
# Enemy spawner
extends Node2D

func spawn_enemy(position: Vector2):
    var enemy = enemy_scene.instantiate()
    enemy.global_position = position
    enemy.add_to_group("enemies")
    add_child(enemy)

# Game manager - damage all enemies
func damage_all_enemies(amount: int):
    get_tree().call_group("enemies", "take_damage", amount)

# Check win condition
func check_victory():
    if get_tree().get_nodes_in_group("enemies").is_empty():
        trigger_victory()
```

### Collectible System
```gdscript
# Collectible item
extends Area2D

func _ready():
    add_to_group("collectibles")
    body_entered.connect(_on_collected)

func _on_collected(body):
    if body.is_in_group("player"):
        # Update UI counter
        get_tree().call_group("ui_managers", "update_collectible_count")
        queue_free()
```

### Pause System
```gdscript
# Game manager
extends Node

func pause_game():
    get_tree().paused = true
    get_tree().set_group("pausable", "process_mode", Node.PROCESS_MODE_DISABLED)

func resume_game():
    get_tree().paused = false
    get_tree().set_group("pausable", "process_mode", Node.PROCESS_MODE_INHERIT)

# Add to pausable objects
func _ready():
    add_to_group("pausable")
```

### Level Cleanup
```gdscript
# Scene transition manager
extends Node

func cleanup_level():
    # Remove all temporary objects
    get_tree().call_group("temporary", "queue_free")
    
    # Reset persistent objects
    get_tree().call_group("persistent", "reset_state")
    
    # Hide UI elements
    get_tree().call_group("level_ui", "visible", false)
```

## Performance Tips

### Use Call Flags for Optimization
```gdscript
# Defer calls to avoid issues during physics processing
get_tree().call_group_flags(
    SceneTree.GROUP_CALL_DEFERRED,
    "enemies", 
    "die"
)

# Reverse call order (useful for cleanup)
get_tree().call_group_flags(
    SceneTree.GROUP_CALL_REVERSE,
    "cleanup_objects",
    "cleanup"
)

# Unique calls only (avoid duplicate calls in same frame)
get_tree().call_group_flags(
    SceneTree.GROUP_CALL_UNIQUE,
    "audio_sources",
    "stop"
)
```

### Efficient Group Management
```gdscript
# Cache group references for frequently accessed groups
@onready var players = get_tree().get_nodes_in_group("player")
@onready var enemies = get_tree().get_nodes_in_group("enemies")

# Use specific group names to avoid conflicts
add_to_group("level1_enemies")  # Instead of just "enemies"
add_to_group("ui_main_menu")    # Instead of just "ui"
```

## Common Patterns

### Singleton Access
```gdscript
# Add singleton to group in _ready()
func _ready():
    add_to_group("game_manager")

# Access from anywhere
var game_manager = get_tree().get_first_node_in_group("game_manager")
```

### Event Broadcasting
```gdscript
# Broadcast events to listeners
func broadcast_game_event(event_name: String, data = null):
    get_tree().call_group("event_listeners", "on_game_event", event_name, data)

# Listener nodes implement on_game_event
func on_game_event(event_name: String, data):
    match event_name:
        "player_died":
            show_death_screen()
        "score_changed":
            update_score_display(data)
```

Groups provide a flexible and efficient way to organize and manage nodes in your Godot projects. They're particularly powerful for game systems that need to operate on multiple objects simultaneously.

