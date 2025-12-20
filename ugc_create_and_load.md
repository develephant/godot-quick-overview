# User-Generated Content (UGC) in Godot 4

## Overview

Godot supports loading user-generated content through multiple methods, each with different capabilities and security considerations.

---

## 1. Resource Files (.tres, .res)

### What Can Be Loaded
- Custom resources (weapon stats, character data, etc.)
- Scenes (.tscn, .scn)
- Images, audio, 3D models
- Any resource type Godot supports

### How to Load

```gdscript
# Load a user-created weapon
var weapon = load("user://mods/custom_weapon.tres") as WeaponData

# Load a custom level scene
var level_scene = load("user://levels/user_level.tscn")
var level_instance = level_scene.instantiate()
add_child(level_instance)
```

### Specs & Limitations
- ✅ **Safe** - Can't execute arbitrary code
- ✅ **Easy to create** - Users can make .tres files in editor
- ❌ **Requires Godot editor** - Users need Godot to create content
- 📁 **File location** - `user://` maps to OS-specific directory

---

## 2. JSON Data Files

### What Can Be Loaded
- Character stats
- Level configurations
- Item databases
- Quest data
- Any structured data

### How to Load

```gdscript
func load_user_item() -> Dictionary:
    var file = FileAccess.open("user://items/sword.json", FileAccess.READ)
    if file:
        var json_string = file.get_as_text()
        var json = JSON.new()
        var result = json.parse(json_string)
        if result == OK:
            return json.data
    return {}

# Example JSON file:
# {
#   "name": "Flaming Sword",
#   "damage": 50,
#   "element": "fire"
# }
```

### Specs & Limitations
- ✅ **Very safe** - Plain text data only
- ✅ **User-friendly** - Can edit in any text editor
- ✅ **Cross-platform compatible**
- ❌ **No visual data** - Can't store scenes/images directly
- 📝 **Format** - Standard JSON format

---

## 3. Image Files (Runtime Loading)

### Supported Formats
- PNG
- JPG/JPEG
- WebP
- BMP (limited support)

### How to Load

```gdscript
func load_user_texture(path: String) -> ImageTexture:
    var image = Image.new()
    var error = image.load(path)
    
    if error == OK:
        return ImageTexture.create_from_image(image)
    else:
        print("Failed to load image: ", path)
        return null

# Usage
var custom_icon = load_user_texture("user://textures/custom_icon.png")
$Sprite2D.texture = custom_icon
```

### Specs & Limitations
- ✅ **Safe** - Image data only
- ✅ **Standard formats** - Users can use any image editor
- ⚠️ **Size limits** - Large images use more memory
- 📏 **Recommended max** - 4096x4096 for textures

---

## 4. GDScript Files (.gd) - ADVANCED

### ⚠️ Security Warning
Loading user GDScript is **DANGEROUS** - it can execute arbitrary code on the user's machine!

### How to Load (If You Must)

```gdscript
# DON'T DO THIS with untrusted content!
var script = load("user://mods/custom_behavior.gd")
var custom_object = script.new()
```

### When It's Acceptable
- ✅ **Local mods only** (not downloaded from internet)
- ✅ **Single-player games**
- ✅ **Developer tools**
- ❌ **Never for multiplayer** (major security risk)
- ❌ **Never from untrusted sources**

---

## 5. PCK/ZIP Archives (Mod Packs)

### What They Are
- **PCK** - Godot's packed scene format
- Can contain multiple resources bundled together

### How to Load

```gdscript
# Load a mod pack
ProjectSettings.load_resource_pack("user://mods/weapons_pack.pck")

# Now you can access resources from the pack
var sword = load("res://weapons/fire_sword.tres")
```

### Specs & Limitations
- ✅ **Efficient** - Multiple files in one archive
- ✅ **Safe** - Pre-compiled resources
- ⚠️ **Requires export** - Users need to export PCK from Godot
- 📦 **Use case** - Best for complex mod packs

---

## User Directory Locations

### Where Files Go

```gdscript
# Get the user data directory
var user_path = OS.get_user_data_dir()
print(user_path)
```

### Platform-Specific Paths

| Platform | Path |
|----------|------|
| **Windows** | `%APPDATA%\Godot\app_userdata\[project_name]` |
| **Linux** | `~/.local/share/godot/app_userdata/[project_name]` |
| **macOS** | `~/Library/Application Support/Godot/app_userdata/[project_name]` |

---

## Complete UGC System Example

```gdscript
# ugc_manager.gd
class_name UGCManager extends Node

const UGC_DIR = "user://user_content/"

func _ready():
    # Create directory if it doesn't exist
    DirAccess.make_dir_recursive_absolute(UGC_DIR)

func load_all_custom_weapons() -> Array[WeaponData]:
    var weapons: Array[WeaponData] = []
    var dir = DirAccess.open(UGC_DIR + "weapons/")
    
    if dir:
        dir.list_dir_begin()
        var file_name = dir.get_next()
        
        while file_name != "":
            if file_name.ends_with(".tres"):
                var weapon = load(UGC_DIR + "weapons/" + file_name)
                if weapon is WeaponData:
                    weapons.append(weapon)
            file_name = dir.get_next()
    
    return weapons

func save_custom_level(level_data: Dictionary, level_name: String) -> void:
    var file = FileAccess.open(UGC_DIR + "levels/" + level_name + ".json", FileAccess.WRITE)
    file.store_string(JSON.stringify(level_data, "\t"))
    file.close()

func load_custom_level(level_name: String) -> Dictionary:
    var file = FileAccess.open(UGC_DIR + "levels/" + level_name + ".json", FileAccess.READ)
    if file:
        var json = JSON.new()
        json.parse(file.get_as_text())
        return json.data
    return {}
```

---

## Best Practices

### ✅ Do's
- **Validate all loaded data** - Check types and ranges
- **Use `user://` path** - OS-independent file access
- **Create clear documentation** - Tell users what formats are supported
- **Implement file size limits** - Prevent memory issues
- **Test with malformed data** - Handle errors gracefully
- **Use JSON for configuration** - Safest option
- **Provide example files** - Help users understand format

### ❌ Don'ts
- **Never load .gd scripts from untrusted sources**
- **Don't assume files exist** - Always check
- **Don't load unlimited file sizes** - Memory overflow risk
- **Don't skip validation** - Bad data can crash your game
- **Don't use `res://` for UGC** - Reserved for game files

---

## Security Levels

| Method | Safety | User-Friendliness | Use Case |
|--------|--------|-------------------|----------|
| **JSON** | 🟢 Very Safe | 🟢 Easy | Stats, configs |
| **.tres Resources** | 🟢 Safe | 🟡 Medium | Items, weapons |
| **Images** | 🟢 Safe | 🟢 Easy | Custom textures |
| **.tscn Scenes** | 🟡 Mostly Safe | 🟡 Medium | Custom levels |
| **PCK Archives** | 🟡 Mostly Safe | 🔴 Hard | Mod packs |
| **GDScript** | 🔴 DANGEROUS | 🟡 Medium | Dev tools only |

---

## Recommended Approach for Beginners

1. **Start with JSON** for data (stats, configs)
2. **Add image loading** for custom textures/icons
3. **Consider .tres resources** if you want rich data
4. **Avoid GDScript loading** unless absolutely necessary

This gives users flexibility while keeping your game secure! 🎮
