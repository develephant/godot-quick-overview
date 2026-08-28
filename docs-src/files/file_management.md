# File Management

## Loading Resources

### `load()` vs `preload()`

| Function | When to use |
|---|---|
| `preload("res://path")` | Path is known at compile time; fast and safe |
| `load("res://path")` | Path is dynamic; loaded when called |

```gd
# Compile time
const ICON = preload("res://icon.png")

# Runtime
var path = "res://data/items/" + item_name + ".tres"
var item = load(path) as ItemData
```

## Loading and Instantiating Scenes

```gd
var scene = load("res://scenes/enemy.tscn") as PackedScene
var enemy = scene.instantiate()
add_child(enemy)
```

```gd
# Shorthand
var player = load("res://scenes/player.tscn").instantiate()
get_tree().current_scene.add_child(player)
```

## Loading JSON and Text Data

```gd
var file = FileAccess.open("res://data/config.json", FileAccess.READ)
if not file:
    return

var json = JSON.new()
var err = json.parse(file.get_as_text())
file.close()

if err == OK:
    var data = json.data
    print(data["title"])
```

## Saving and Loading Resources

```gd
# Create and save a resource
var weapon = WeaponData.new()
weapon.weapon_name = "Iron Axe"
weapon.damage = 15

DirAccess.make_dir_recursive_absolute("user://weapons")
ResourceSaver.save(weapon, "user://weapons/iron_axe.tres")

# Load it back
var loaded = load("user://weapons/iron_axe.tres") as WeaponData
```

### ConfigFile for settings

```gd
var config = ConfigFile.new()
config.set_value("audio", "volume", 0.8)
config.save("user://settings.cfg")
```

## Threaded Loading

```gd
var path := "res://scenes/big_level.tscn"
ResourceLoader.load_threaded_request(path)

while ResourceLoader.load_threaded_get_status(path) == ResourceLoader.THREAD_LOAD_IN_PROGRESS:
    await get_tree().process_frame

var scene = ResourceLoader.load_threaded_get(path) as PackedScene
```

## Hidden / Useful Directories

| Path | Purpose |
|---|---|
| `res://` | Project files (read-only in exported games) |
| `user://` | User data, saves, and settings |
| `OS.get_user_data_dir()` | Absolute OS path that `user://` maps to |
| `.godot/` | Imported assets, shader cache, and editor state (do not commit) |
| `.godot/imported/` | Cached imported resources |
| `.godot/editor/` | Editor layout and state |

!!! tip "Find your user data path"
    `OS.get_user_data_dir()` is great for debugging. It returns the absolute OS path that `user://` maps to.

## Tips and Pitfalls

```gd
# load() returns a Resource. Cast it for typed access.
var weapon = load("res://sword.tres") as WeaponData

# Instantiating a PackedScene is not the same as loading it
var scene = load("res://enemy.tscn")      # PackedScene
var enemy = scene.instantiate()            # Node instance

# ResourceSaver overwrites existing files
# Make sure the directory exists first
DirAccess.make_dir_recursive_absolute("user://weapons")
ResourceSaver.save(weapon, "user://weapons/iron_axe.tres")
```

__Quick Summary:__

- `load()` and `preload()` bring assets into memory.
- Loaded `PackedScene`s must be `instantiate()`-d to become nodes.
- `ResourceSaver.save()` persists `Resource` objects.
- `user://` is for writable runtime data; `res://` is for shipped files.
