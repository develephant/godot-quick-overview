# Working with Files

## FileAccess Quick Reference

Godot 4 uses `FileAccess` for reading and writing files. It works with `res://` (project files) and `user://` (user data) paths.

| Mode | Use |
|---|---|
| `FileAccess.READ` | Open for reading |
| `FileAccess.WRITE` | Create or truncate and write |
| `FileAccess.READ_WRITE` | Open existing file for both |

### Reading a text file

```gd
var file = FileAccess.open("res://data/level_list.txt", FileAccess.READ)
if file:
    var text = file.get_as_text()
    file.close()
```

### Writing text

```gd
var file = FileAccess.open("user://log.txt", FileAccess.WRITE)
if file:
    file.store_line("Game started")
    file.close()
```

### Reading line by line

```gd
var file = FileAccess.open("user://save_data.txt", FileAccess.READ)
if file:
    while not file.eof_reached():
        var line = file.get_line()
        print(line)
    file.close()
```

### Binary files

```gd
var file = FileAccess.open("user://snapshot.bin", FileAccess.WRITE)
file.store_32(12345)
file.store_float(1.5)
file.store_buffer(image_data)
file.close()
```

## DirAccess: Traversing Directories

### List files

```gd
var dir = DirAccess.open("res://data/items")
if dir:
    dir.list_dir_begin()
    var file_name = dir.get_next()
    while file_name != "":
        if not dir.current_is_dir():
            print(file_name)
        file_name = dir.get_next()
    dir.list_dir_end()
```

### Iterate a folder of `.tres` files

```gd
var dir = DirAccess.open("res://data/weapons")
if dir:
    dir.list_dir_begin()
    var file_name = dir.get_next()

    while file_name != "":
        if file_name.ends_with(".tres"):
            var path = "res://data/weapons/" + file_name
            var weapon = load(path) as WeaponData
            if weapon:
                print(weapon.weapon_name)
        file_name = dir.get_next()

    dir.list_dir_end()
```

### Create directories

```gd
DirAccess.make_dir_recursive_absolute("user://saves/backup")
```

## Preloading Assets

Use `preload()` for known paths at compile time. It is faster and safer than `load()`.

```gd
const BULLET_SCENE = preload("res://scenes/bullet.tscn")
const COIN_TEXTURE = preload("res://sprites/coin.png")

func spawn_bullet():
    var bullet = BULLET_SCENE.instantiate()
    add_child(bullet)
```

`preload()` only works with string literal paths inside `res://`.

## Common FileAccess Methods

| Method | Description |
|---|---|
| `FileAccess.file_exists(path)` | Check if a file exists |
| `file.get_as_text()` | Read whole file as a `String` |
| `file.get_line()` | Read one line |
| `file.get_csv_line()` | Read one CSV row |
| `file.store_line(text)` | Write a line with a newline |
| `file.store_string(text)` | Write text without a newline |
| `file.get_position()` / `file.get_length()` | Track read progress |
| `file.seek(position)` | Jump to a byte offset |
| `file.close()` | Close the file |

## Tips and Pitfalls

```gd
# Always check FileAccess.open returned a valid object
var file = FileAccess.open("user://save.json", FileAccess.READ)
if not file:
    return

# Always close when done
file.close()

# Use user:// for anything written at runtime
# res:// is read-only in exported games
```

__Quick Summary:__

- `FileAccess` is the main class for file read/write.
- `DirAccess` lists, creates, and copies directories.
- `preload("res://...")` is for compile-time known assets.
- Use `user://` for saved data and `res://` for shipped assets.
