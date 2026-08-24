# Scene Manager

An example Scene manager.

```gd
extends Node

signal scene_changed(new_scene: String)

var current_scene: Node = null

func _ready() -> void:
	current_scene = get_tree().current_scene
	print("Game Manager ready.")

# Change to a new scene
func change_scene(new_scene_path: String) -> void:
	call_deferred("_deferred_scene_change", new_scene_path)

func _deferred_scene_change(new_scene_path: String) -> void:
	# Clean up before changing scenes
	if current_scene:
		current_scene.queue_free()

	# Now load the new scene
	var new_scene = load(new_scene_path).instantiate()

	get_tree().root.add_child(new_scene)
	get_tree().current_scene = new_scene
	current_scene = new_scene

	scene_changed.emit(new_scene_path)


# Restart the current scene
func restart_scene() -> void:
	if current_scene:
		var scene_path = current_scene.scene_file_path
		change_scene(scene_path)

# Get the name of the current scene
func get_current_scene_name() -> String:
	if current_scene:
		return current_scene.scene_file_path.get_file().get_basename()
	return ""

# Pause the game
func pause_game() -> void:
	get_tree().paused = true

# Resume the game
func resume_game() -> void:
	get_tree().paused = false

# Toggle pause state
func toggle_pause() -> void:
	get_tree().paused = not get_tree().paused

# Check if the game is paused
func is_game_paused() -> bool:
	return get_tree().paused

# Quit the game
func quit_game() -> void:
	get_tree().quit()
```
