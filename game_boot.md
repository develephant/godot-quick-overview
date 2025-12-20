
# Game Boot

## Project Structure

```
 autoload/ - All singleton scripts
   ├── game_manager.gd
   ├── save_manager.gd
   ├── audio_manager.gd
   └── economy_manager.gd
 scenes/
   ├── boot.tscn (first scene)
   ├── main_menu.tscn
   └── game.tscn
```

## boot.gd

The first `scene` that loads.

```php
# ============================================
# boot.gd - First scene that loads
# ============================================
extends Node

func _ready():
    # 1. Initialize autoloads that need setup:
    SaveManager.load_settings()
    AudioManager.set_volume(SaveManager.settings.volume)
    
    # 2. Show loading screen:
    $LoadingScreen.show()
    
    # 3. Load resources:
    await preload_resources()
    
    # 4. Transition to main menu:
    get_tree().change_scene_to_file("res://scenes/main_menu.tscn")

func preload_resources():
    # Load heavy assets:
    ResourceLoader.load_threaded_request("res://assets/big_texture.png")
    # ... more resources
    await get_tree().create_timer(0.5).timeout  # Fake delay for demonstration
```

## game_manager.gd

Autoload singleton

```php
# ============================================
# game_manager.gd - AutoLoad singleton
# ============================================
extends Node

signal scene_changed(new_scene: String)

var current_scene: Node = null

func _ready():
    # Get reference to first scene:
    current_scene = get_tree().current_scene
    print("GameManager initialized")

func change_scene(scene_path: String):
    # Deferred call to avoid issues:
    call_deferred("_deferred_change_scene", scene_path)

func _deferred_change_scene(scene_path: String):
    # Clean up current scene:
    if current_scene:
        current_scene.free()
    
    # Load new scene:
    var new_scene = load(scene_path).instantiate()
    get_tree().root.add_child(new_scene)
    get_tree().current_scene = new_scene
    current_scene = new_scene
    
    scene_changed.emit(scene_path)
```

## Add `autoloads` to Settings


Navigate to __Project Settings > Autoload__ in the editor.

Add these in order (top to bottom matters):

    1. SaveManager (loads first)
    2. GameManager
    3. AudioManager
    4. EconomyManager (depends on GameManager signals)
