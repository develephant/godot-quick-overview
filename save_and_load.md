
# Saving and Loading

## save_manager.gd

```php
# ============================================
# save_manager.gd - AutoLoad singleton
# ============================================
extends Node

const SAVE_PATH = "user://savegame.save"

signal game_saved
signal game_loaded

func save_game():
    var save_file = FileAccess.open(SAVE_PATH, FileAccess.WRITE)
    
    # Collect save data from all saveable nodes:
    var save_nodes = get_tree().get_nodes_in_group("persist")
    for node in save_nodes:
        # Check node has save method:
        if !node.has_method("save"):
            continue
        
        # Get node data:
        var node_data = node.call("save")
        
        # Store scene path for recreation:
        node_data["scene_path"] = node.scene_file_path
        node_data["parent_path"] = node.get_parent().get_path()
        
        # Write as JSON line:
        var json_string = JSON.stringify(node_data)
        save_file.store_line(json_string)
    
    save_file.close()
    game_saved.emit()
    print("Game saved!")

func load_game():
    if not FileAccess.file_exists(SAVE_PATH):
        print("No save file found")
        return
    
    var save_file = FileAccess.open(SAVE_PATH, FileAccess.READ)
    
    # Clear current game:
    var save_nodes = get_tree().get_nodes_in_group("persist")
    for node in save_nodes:
        node.queue_free()
    
    # Load saved nodes:
    while save_file.get_position() < save_file.get_length():
        var json_string = save_file.get_line()
        var json = JSON.new()
        var parse_result = json.parse(json_string)
        
        if parse_result != OK:
            continue
        
        var node_data = json.data
        
        # Recreate node:
        var new_node = load(node_data["scene_path"]).instantiate()
        get_node(node_data["parent_path"]).add_child(new_node)
        
        # Restore state:
        if new_node.has_method("load_data"):
            new_node.load_data(node_data)
    
    save_file.close()
    game_loaded.emit()
    print("Game loaded!")
```

__Example:__

```php
# ============================================
# Example saveable node
# ============================================
extends CharacterBody2D

@export var player_name: String = "Hero"
var gold: int = 100
var level: int = 1

func _ready():
    add_to_group("persist")

func save() -> Dictionary:
    return {
        "pos_x": position.x,
        "pos_y": position.y,
        "gold": gold,
        "level": level,
        "player_name": player_name
    }

func load_data(data: Dictionary):
    position.x = data["pos_x"]
    position.y = data["pos_y"]
    gold = data["gold"]
    level = data["level"]
    player_name = data["player_name"]
```

## Save Settings

```php
# ============================================
# Settings save (simpler, single file)
# ============================================
extends Node

const SETTINGS_PATH = "user://settings.cfg"

var settings = {
    "volume": 0.8,
    "fullscreen": false,
    "difficulty": 1
}

func save_settings():
    var config = ConfigFile.new()
    config.set_value("audio", "volume", settings.volume)
    config.set_value("video", "fullscreen", settings.fullscreen)
    config.set_value("game", "difficulty", settings.difficulty)
    config.save(SETTINGS_PATH)

func load_settings():
    var config = ConfigFile.new()
    var err = config.load(SETTINGS_PATH)
    if err != OK:
        return
    
    settings.volume = config.get_value("audio", "volume", 0.8)
    settings.fullscreen = config.get_value("video", "fullscreen", false)
    settings.difficulty = config.get_value("game", "difficulty", 1)
```
