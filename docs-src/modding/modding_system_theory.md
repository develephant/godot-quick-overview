# Modding System Theory - Godot 4

## Overview

Godot 4 supports multiple approaches to modding, from in-game editors to standalone external tools. This guide covers the theory and implementation of mod systems.

---

## Mod Creation Approaches

### 1. In-Game Editor (Built Into Your Game)

**What it is:** A mod creation tool built directly into your game's UI.

**Advantages:**
- ✅ Players don't need any external software
- ✅ Context-aware (can preview in actual game)
- ✅ Instant testing and iteration
- ✅ Streamlined workflow
- ✅ Can use game's existing assets

**Disadvantages:**
- ❌ Takes development time to build
- ❌ UI space limited to game window
- ❌ More complex to implement
- ❌ Harder to update independently

**Best For:**
- Simple mod types (character stats, items, basic levels)
- Games where modding is a core feature
- Mobile/console games (no file system access)

```gdscript
# ============================================
# IN-GAME WEAPON CREATOR EXAMPLE
# ============================================
extends Control

@onready var weapon_name_input: LineEdit = $Panel/WeaponName
@onready var damage_slider: HSlider = $Panel/Damage
@onready var speed_slider: HSlider = $Panel/Speed
@onready var preview_sprite: Sprite2D = $PreviewArea/Sprite

func _on_create_button_pressed() -> void:
    var weapon = WeaponData.new()
    weapon.weapon_name = weapon_name_input.text
    weapon.damage = int(damage_slider.value)
    weapon.attack_speed = speed_slider.value
    weapon.icon = preview_sprite.texture
    
    # Save to user directory
    var save_path = "user://mods/weapons/" + weapon.weapon_name.to_snake_case() + ".tres"
    DirAccess.make_dir_recursive_absolute("user://mods/weapons/")
    
    if ResourceSaver.save(weapon, save_path) == OK:
        show_success_popup("Weapon created successfully!")
        # Immediately available in game
        GameManager.reload_custom_weapons()

func _on_test_button_pressed() -> void:
    # Spawn enemy to test weapon on
    spawn_test_dummy()
```

---

### 2. External Standalone Editor (Separate Godot Project)

**What it is:** A completely separate Godot application dedicated to creating mods.

**Advantages:**
- ✅ More screen space for complex editing
- ✅ Can be updated independently from main game
- ✅ More powerful tools/features
- ✅ Can run on different platforms
- ✅ Professional-grade features without bloating main game

**Disadvantages:**
- ❌ Users need to download separate tool
- ❌ More work to maintain two projects
- ❌ Requires exporting another application

**Best For:**
- Complex mod types (full campaigns, detailed levels)
- Games with dedicated modding communities
- Professional/advanced mod creation
- PC games

```gdscript
# ============================================
# EXTERNAL LEVEL EDITOR (Separate Project)
# ============================================
# This runs as its own standalone application

extends Node2D

var current_level: LevelData
var selected_tool: String = "spawn"
var enemy_spawn_points: Array[Vector2] = []
var item_positions: Array[Vector2] = []

func _ready():
    current_level = LevelData.new()
    setup_tool_palette()
    setup_preview_area()

func _input(event):
    if event is InputEventMouseButton and event.pressed:
        var click_pos = get_global_mouse_position()
        
        match selected_tool:
            "spawn":
                add_enemy_spawn(click_pos)
            "item":
                add_item_position(click_pos)
            "wall":
                add_wall_segment(click_pos)

func add_enemy_spawn(pos: Vector2):
    current_level.enemy_spawn_points.append(pos)
    # Visual marker in editor
    var marker = preload("res://editor/spawn_marker.tscn").instantiate()
    marker.position = pos
    $Markers.add_child(marker)

func save_level():
    var dialog = FileDialog.new()
    dialog.file_mode = FileDialog.FILE_MODE_SAVE_FILE
    dialog.filters = ["*.tres ; Level Resource"]
    dialog.file_selected.connect(_on_save_location_selected)
    add_child(dialog)
    dialog.popup_centered()

func _on_save_location_selected(path: String):
    current_level.level_name = path.get_file().get_basename()
    
    if ResourceSaver.save(current_level, path) == OK:
        print("Level saved: ", path)
        show_notification("Level saved successfully!\nPlace in game's user://levels/ folder")
    else:
        show_error("Failed to save level!")

func export_for_game():
    # Copy to a location users can easily find
    var export_path = OS.get_system_dir(OS.SYSTEM_DIR_DOCUMENTS) + "/MyGame/Mods/Levels/"
    DirAccess.make_dir_recursive_absolute(export_path)
    
    var filename = current_level.level_name + ".tres"
    ResourceSaver.save(current_level, export_path + filename)
    
    show_notification("Level exported to:\n" + export_path)
```

---

### 3. Hybrid Approach

**What it is:** Simple in-game editor + advanced external tool.

**Example:**
- In-game: Quick weapon/character creator
- External: Complex level editor, campaign builder

```gdscript
# ============================================
# HYBRID: Simple in-game + External editor
# ============================================

# IN-GAME: Quick item tweaker
func quick_modify_item(item: ItemData):
    var popup = preload("res://ui/quick_item_editor.tscn").instantiate()
    popup.load_item(item)
    add_child(popup)
    # Make quick stat changes, save back

# EXTERNAL EDITOR: Import and enhance
func import_from_advanced_editor():
    var dir = DirAccess.open("user://mods/advanced/")
    # Load complex mods created in external tool
    # These have features not available in-game
```

---

## Creating .tres Files Programmatically

### The Core Concept

**You don't need the Godot editor to create `.tres` files!** They can be created entirely in code at runtime.

```gdscript
# ============================================
# CREATING RESOURCES IN CODE
# ============================================

# 1. Create the resource instance
var weapon = WeaponData.new()

# 2. Set properties
weapon.weapon_name = "Fire Sword"
weapon.damage = 50
weapon.element = "fire"

# 3. Save as .tres file
ResourceSaver.save(weapon, "user://mods/fire_sword.tres")

# 4. Load it back later
var loaded_weapon = load("user://mods/fire_sword.tres") as WeaponData
```

### Advanced: Embedding User Images

```gdscript
# ============================================
# LOAD USER IMAGE AND EMBED IN .TRES
# ============================================
func create_character_with_custom_sprite():
    var character = CharacterData.new()
    character.name = "Custom Hero"
    
    # Let user pick an image
    var image_path = "user://uploads/hero_sprite.png"
    var image = Image.new()
    
    if image.load(image_path) == OK:
        # Convert to texture and embed
        character.sprite_texture = ImageTexture.create_from_image(image)
    
    # Save everything (including image data)
    ResourceSaver.save(character, "user://mods/characters/custom_hero.tres")
    
    # The .tres file now contains the image data!
```

---

## Complete Mod System Architecture

```gdscript
# ============================================
# MOD MANAGER - Game Side
# ============================================
class_name ModManager extends Node

signal mods_loaded
signal mod_failed(mod_name: String, error: String)

const MOD_DIR = "user://mods/"

var loaded_weapons: Array[WeaponData] = []
var loaded_levels: Array[LevelData] = []
var loaded_characters: Array[CharacterData] = []

func _ready():
    create_mod_directories()
    load_all_mods()

func create_mod_directories():
    DirAccess.make_dir_recursive_absolute(MOD_DIR + "weapons/")
    DirAccess.make_dir_recursive_absolute(MOD_DIR + "levels/")
    DirAccess.make_dir_recursive_absolute(MOD_DIR + "characters/")

func load_all_mods():
    loaded_weapons = load_mods_from_directory(MOD_DIR + "weapons/", WeaponData)
    loaded_levels = load_mods_from_directory(MOD_DIR + "levels/", LevelData)
    loaded_characters = load_mods_from_directory(MOD_DIR + "characters/", CharacterData)
    
    print("Loaded %d weapon mods" % loaded_weapons.size())
    print("Loaded %d level mods" % loaded_levels.size())
    print("Loaded %d character mods" % loaded_characters.size())
    
    mods_loaded.emit()

func load_mods_from_directory(dir_path: String, expected_type) -> Array:
    var mods = []
    var dir = DirAccess.open(dir_path)
    
    if not dir:
        return mods
    
    dir.list_dir_begin()
    var file_name = dir.get_next()
    
    while file_name != "":
        if file_name.ends_with(".tres"):
            var full_path = dir_path + file_name
            var resource = load(full_path)
            
            # Validate type
            if is_instance_of(resource, expected_type):
                # Validate content
                if validate_mod(resource):
                    mods.append(resource)
                else:
                    mod_failed.emit(file_name, "Validation failed")
            else:
                mod_failed.emit(file_name, "Wrong resource type")
        
        file_name = dir.get_next()
    
    return mods

func validate_mod(mod_resource: Resource) -> bool:
    # Prevent malicious or broken mods
    if mod_resource is WeaponData:
        var weapon = mod_resource as WeaponData
        # Check for reasonable values
        if weapon.damage < 0 or weapon.damage > 10000:
            return false
        if weapon.attack_speed < 0.1 or weapon.attack_speed > 100:
            return false
        return true
    
    # Add validation for other types
    return true

func get_weapon_by_name(weapon_name: String) -> WeaponData:
    for weapon in loaded_weapons:
        if weapon.weapon_name == weapon_name:
            return weapon
    return null
```

---

## Resource Data Structures

```gdscript
# ============================================
# WEAPON RESOURCE
# ============================================
class_name WeaponData extends Resource

@export_group("Identity")
@export var weapon_name: String = "Unnamed Weapon"
@export_multiline var description: String = ""
@export var icon: Texture2D

@export_group("Stats")
@export var damage: int = 10
@export var attack_speed: float = 1.0
@export_range(0, 1) var critical_chance: float = 0.1
@export var element: Element = Element.NONE

@export_group("Requirements")
@export var required_level: int = 1
@export var cost: int = 100

enum Element { NONE, FIRE, ICE, LIGHTNING, POISON }

func get_dps() -> float:
    return damage * attack_speed

# ============================================
# LEVEL RESOURCE
# ============================================
class_name LevelData extends Resource

@export_group("Info")
@export var level_name: String = "Untitled Level"
@export_multiline var description: String = ""
@export var author: String = ""
@export var thumbnail: Texture2D

@export_group("Layout")
@export var enemy_spawn_points: Array[Vector2] = []
@export var item_positions: Array[Vector2] = []
@export var player_start_position: Vector2 = Vector2.ZERO
@export var wall_segments: Array[PackedVector2Array] = []

@export_group("Gameplay")
@export var background_music: AudioStream
@export var time_limit: float = 0.0  # 0 = no limit
@export var difficulty: Difficulty = Difficulty.NORMAL

enum Difficulty { EASY, NORMAL, HARD, EXPERT }

# ============================================
# CHARACTER RESOURCE
# ============================================
class_name CharacterData extends Resource

@export_group("Identity")
@export var character_name: String = "Hero"
@export var sprite_texture: Texture2D
@export var portrait: Texture2D

@export_group("Base Stats")
@export var max_health: int = 100
@export var move_speed: float = 200.0
@export var jump_power: float = 500.0

@export_group("Starting Equipment")
@export var starting_weapon: WeaponData
@export var starting_items: Array[ItemData] = []

@export_group("Abilities")
@export var can_double_jump: bool = false
@export var can_dash: bool = false
@export var special_ability: String = ""
```

---

## External Editor Structure

```gdscript
# ============================================
# EXTERNAL EDITOR PROJECT STRUCTURE
# ============================================

# MyGameEditor/
# ├── main.tscn              # Main editor window
# ├── tool_palette.tscn      # Tool selection UI
# ├── property_panel.tscn    # Edit resource properties
# ├── preview_area.tscn      # Preview the mod
# ├── export_dialog.tscn     # Export settings
# └── scripts/
#     ├── editor_main.gd
#     ├── resource_creator.gd
#     └── validation.gd

# editor_main.gd
extends Control

var current_resource: Resource
var resource_type: String = "weapon"  # weapon, level, character

func _ready():
    setup_ui()
    setup_menus()

func new_mod(type: String):
    match type:
        "weapon":
            current_resource = WeaponData.new()
        "level":
            current_resource = LevelData.new()
        "character":
            current_resource = CharacterData.new()
    
    load_resource_into_editor(current_resource)

func load_mod(path: String):
    current_resource = load(path)
    load_resource_into_editor(current_resource)

func save_mod(path: String):
    if validate_resource(current_resource):
        ResourceSaver.save(current_resource, path)
        show_notification("Mod saved!")
    else:
        show_error("Validation failed! Check your values.")

func export_to_game():
    # Help users get mod to right location
    var user_docs = OS.get_system_dir(OS.SYSTEM_DIR_DOCUMENTS)
    var export_path = user_docs + "/MyGame/Mods/"
    
    # Create instructions file
    var readme = FileAccess.open(export_path + "README.txt", FileAccess.WRITE)
    readme.store_string("""
    HOW TO INSTALL MODS:
    
    1. Copy the .tres files to your game's mod folder:
       Windows: %APPDATA%/Godot/app_userdata/MyGame/mods/
       Linux: ~/.local/share/godot/app_userdata/MyGame/mods/
       macOS: ~/Library/Application Support/Godot/app_userdata/MyGame/mods/
    
    2. Launch the game
    3. Your mods will appear in the custom content menu!
    """)
    readme.close()
    
    # Copy the mod file
    var filename = current_resource.get("level_name", "mod") + ".tres"
    ResourceSaver.save(current_resource, export_path + filename)
    
    OS.shell_show_in_file_manager(export_path)
```

---

## Mod Distribution

### Option 1: Manual File Sharing

Users share `.tres` files directly.

**Instructions to users:**
1. Download the `.tres` file
2. Place in `[User Data Dir]/mods/weapons/`
3. Launch game

### Option 2: Auto-Install System

```gdscript
# ============================================
# MOD INSTALLER
# ============================================
extends Node

func install_mod_from_file(mod_file_path: String) -> bool:
    # Detect mod type by loading and checking
    var mod = load(mod_file_path)
    
    var destination = ""
    if mod is WeaponData:
        destination = "user://mods/weapons/"
    elif mod is LevelData:
        destination = "user://mods/levels/"
    elif mod is CharacterData:
        destination = "user://mods/characters/"
    else:
        return false
    
    # Copy to correct location
    var filename = mod_file_path.get_file()
    DirAccess.copy_absolute(mod_file_path, destination + filename)
    
    # Reload mods
    ModManager.load_all_mods()
    
    return true
```

### Option 3: Workshop Integration

For Steam Workshop or similar platforms:

```gdscript
# ============================================
# WORKSHOP INTEGRATION (Pseudocode)
# ============================================
func subscribe_to_mod(workshop_id: int):
    # Steam handles download
    Steam.subscribe_item(workshop_id)

func _on_item_installed(item_id: int):
    var mod_path = Steam.get_item_install_info(item_id).folder
    # Load .tres files from workshop folder
    install_workshop_mods(mod_path)
```

---

## Best Practices

### For Game Developers

```gdscript
# ============================================
# VALIDATION IS CRITICAL
# ============================================
func validate_weapon(weapon: WeaponData) -> bool:
    # Prevent exploits
    if weapon.damage < 1 or weapon.damage > 999:
        return false
    
    if weapon.attack_speed < 0.1 or weapon.attack_speed > 10:
        return false
    
    # Prevent inappropriate content (if needed)
    if contains_profanity(weapon.weapon_name):
        return false
    
    return true

# ============================================
# VERSION COMPATIBILITY
# ============================================
class_name WeaponData extends Resource

@export var mod_version: int = 1  # Track mod format version

func is_compatible_with_game() -> bool:
    return mod_version <= GameVersion.MOD_FORMAT_VERSION
```

### For Mod Creators

**Document the resource structure:**

```gdscript
# weapon_data.gd
class_name WeaponData extends Resource

## The displayed name of the weapon
@export var weapon_name: String = "Unnamed Weapon"

## Base damage per hit (1-999)
@export_range(1, 999) var damage: int = 10

## Attacks per second (0.1-10.0)
@export_range(0.1, 10.0) var attack_speed: float = 1.0
```

---

## Comparison Table

| Approach | Complexity | User-Friendly | Power | Best For |
|----------|------------|---------------|-------|----------|
| **In-Game Editor** | 🟡 Medium | 🟢 Very Easy | 🟡 Limited | Simple mods, casual players |
| **External Editor** | 🔴 High | 🟡 Moderate | 🟢 Full | Complex mods, dedicated community |
| **Hybrid** | 🔴 Very High | 🟢 Flexible | 🟢 Full | Professional modding support |
| **JSON Only** | 🟢 Easy | 🟢 Easy | 🔴 Very Limited | Stats/config only |

---

## Summary

**Yes, you can create an external Godot editor!**

✅ Separate Godot project = standalone mod editor  
✅ Create `.tres` files programmatically (no main editor needed)  
✅ Users don't need Godot to use mods  
✅ Best of both worlds: simple in-game + powerful external  

**Recommended Workflow:**
1. Define your `Resource` classes in main game
2. Export those scripts to editor project (or keep in sync)
3. Build editor UI with creation/preview tools
4. Save mods as `.tres` files
5. Users drop `.tres` files in game's mod folder
6. Game loads and validates mods at runtime

**This is how professional modding tools work!** 🎮🔧
