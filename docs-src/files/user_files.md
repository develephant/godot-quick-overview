# Working with User Files

## The `user://` Directory

`user://` is the recommended place to store anything your game writes at runtime: saves, settings, logs, screenshots, and downloaded content. It maps to an OS-specific folder, so it works across Windows, macOS, Linux, Android, and iOS.

```gd
print(OS.get_user_data_dir())
# Windows example: C:/Users/Name/AppData/Roaming/Godot/app_userdata/MyGame
```

!!! tip "Why not `res://`?"
    `res://` points at your project files. In an exported build it is read-only, so always write to `user://`.

## Loading and Saving User Data

### Simple text file

```gd
const SETTINGS_PATH = "user://settings.txt"

func save_volume(value: float):
    var file = FileAccess.open(SETTINGS_PATH, FileAccess.WRITE)
    if file:
        file.store_line(str(value))
        file.close()

func load_volume() -> float:
    if not FileAccess.file_exists(SETTINGS_PATH):
        return 1.0
    var file = FileAccess.open(SETTINGS_PATH, FileAccess.READ)
    if not file:
        return 1.0
    var value = file.get_line().to_float()
    file.close()
    return value
```

### ConfigFile for settings

```gd
const SETTINGS_PATH = "user://settings.cfg"

func save_settings():
    var config = ConfigFile.new()
    config.set_value("audio", "volume", 0.8)
    config.set_value("video", "fullscreen", true)
    config.save(SETTINGS_PATH)

func load_settings():
    var config = ConfigFile.new()
    var err = config.load(SETTINGS_PATH)
    if err != OK:
        return
    var volume = config.get_value("audio", "volume", 1.0)
    var fullscreen = config.get_value("video", "fullscreen", false)
```

### Saving JSON

```gd
const SAVE_PATH = "user://save.json"

func save_game(state: Dictionary):
    var file = FileAccess.open(SAVE_PATH, FileAccess.WRITE)
    if not file:
        return
    file.store_string(JSON.stringify(state))
    file.close()

func load_game() -> Dictionary:
    if not FileAccess.file_exists(SAVE_PATH):
        return {}
    var file = FileAccess.open(SAVE_PATH, FileAccess.READ)
    if not file:
        return {}
    var json = JSON.new()
    json.parse(file.get_as_text())
    file.close()
    return json.data
```

## Common Use Cases

| Use case | Suggested path / pattern |
|---|---|
| Save games | `user://saves/slot1.save` |
| Settings | `user://settings.cfg` |
| Logs | `user://logs/session.log` |
| Screenshots | `user://screenshots/` + timestamp |
| UGC / mods | `user://mods/` |

```gd
# Screenshot example
var img = get_viewport().get_texture().get_image()
var time = Time.get_datetime_string_from_system().replace(":", "-")
DirAccess.make_dir_recursive_absolute("user://screenshots")
img.save_png("user://screenshots/shot_" + time + ".png")
```

## Creating User Directories

```gd
DirAccess.make_dir_recursive_absolute("user://saves")
DirAccess.make_dir_recursive_absolute("user://mods/weapons")
```

## Tips and Pitfalls

```gd
# Create the directory before writing
DirAccess.make_dir_recursive_absolute("user://saves")

# Check a file exists before loading
if not FileAccess.file_exists("user://save.json"):
    return

# user:// is created automatically on first write
# Use OS.get_user_data_dir() to find the real folder for debugging
```

__Quick Summary:__

- Use `user://` for all runtime writes.
- `FileAccess`, `ConfigFile`, and `JSON` cover most save/load needs.
- Create missing directories with `DirAccess.make_dir_recursive_absolute`.
- `OS.get_user_data_dir()` reveals the actual OS path.
