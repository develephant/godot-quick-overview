# Custom Resources (.tres Files)

## What are .tres Files?

`.tres` files are **text-based resource files** that store reusable data instances. Think of them like **structs or data templates** that you can create in the editor without writing code.

**File Types:**

- `.tres` - Text format (human-readable, Git-friendly)
- `.res` - Binary format (smaller, faster, not readable)

## Creating Custom Resources

### Step 1: Define Resource Class

```gd
# ============================================
# WEAPON RESOURCE
# ============================================
class_name WeaponData extends Resource

@export_group("Basic Info")
@export var weapon_name: String = "Unnamed Weapon"
@export var description: String = ""
@export var icon: Texture2D

@export_group("Stats")
@export var damage: int = 10
@export var attack_speed: float = 1.0
@export var critical_chance: float = 0.1
@export var durability: int = 100

@export_group("Requirements")
@export var required_level: int = 1
@export var cost: int = 100

func get_dps() -> float:
    return damage * attack_speed

func is_usable_by_player(player_level: int) -> bool:
    return player_level >= required_level
```

### Step 2: Create .tres Files in Editor

**Method 1: FileSystem Panel**

1. Right-click in FileSystem
2. Select "New Resource"
3. Search and select your custom class (e.g., "WeaponData")
4. Save as `sword.tres`

**Method 2: Inspector**

1. Select a node
2. Add `@export var weapon: WeaponData` to script
3. In Inspector, click dropdown next to the property
4. Select "New WeaponData"
5. Click the save icon to save as `.tres`

### Step 3: Configure Values in Inspector

Click on the `.tres` file in FileSystem to edit its values in the Inspector panel.

## Using Resources in Scripts

```gd
# ============================================
# USING .TRES IN CODE
# ============================================
extends Node2D

# Assign in Inspector (drag .tres file)
@export var weapon: WeaponData

func _ready():
    if weapon:
        print("Weapon: ", weapon.weapon_name)
        print("Damage: ", weapon.damage)
        print("DPS: ", weapon.get_dps())

# Load dynamically
func load_weapon(weapon_path: String):
    var weapon_data = load(weapon_path) as WeaponData
    print(weapon_data.weapon_name)

# Preload at compile time
const IRON_SWORD = preload("res://data/weapons/iron_sword.tres")

func _ready():
    print(IRON_SWORD.damage)  # 10
```

## Common Use Cases

### 1. Item Database

```gd
# ============================================
# ITEM RESOURCE
# ============================================
class_name ItemData extends Resource

@export var item_name: String
@export var icon: Texture2D
@export var stack_size: int = 1
@export var value: int = 1
@export var rarity: Rarity = Rarity.COMMON

enum Rarity { COMMON, UNCOMMON, RARE, EPIC, LEGENDARY }

# Create multiple .tres files:
# - health_potion.tres
# - mana_potion.tres
# - gold_coin.tres
# Each with different values
```

### 2. Enemy Configurations

```gd
# ============================================
# ENEMY RESOURCE
# ============================================
class_name EnemyData extends Resource

@export_group("Identity")
@export var enemy_name: String = "Enemy"
@export var sprite: Texture2D

@export_group("Combat Stats")
@export var max_health: int = 50
@export var damage: int = 10
@export var move_speed: float = 100.0
@export var attack_range: float = 50.0

@export_group("AI Behavior")
@export var detection_range: float = 200.0
@export var aggression: float = 0.5
@export var flee_health_percent: float = 0.2

@export_group("Rewards")
@export var experience: int = 25
@export var gold_drop_min: int = 10
@export var gold_drop_max: int = 50
@export var loot_table: Array[ItemData] = []

# Create .tres files:
# - goblin.tres (weak, fast)
# - orc.tres (strong, slow)
# - dragon.tres (very strong, high rewards)
```

```gd
# ============================================
# USING ENEMY DATA
# ============================================
extends CharacterBody2D

@export var enemy_data: EnemyData
var current_health: int

func _ready():
    if enemy_data:
        current_health = enemy_data.max_health
        $Sprite2D.texture = enemy_data.sprite
        $NameLabel.text = enemy_data.enemy_name

func take_damage(amount: int):
    current_health -= amount
    
    if current_health <= 0:
        die()
    elif current_health < enemy_data.max_health * enemy_data.flee_health_percent:
        start_fleeing()

func die():
    # Drop loot based on enemy_data
    drop_gold(randi_range(enemy_data.gold_drop_min, enemy_data.gold_drop_max))
    drop_loot()
```

### 3. Dialogue System

```gd
# ============================================
# DIALOGUE RESOURCE
# ============================================
class_name DialogueData extends Resource

@export var character_name: String
@export var portrait: Texture2D
@export var dialogue_lines: Array[String] = []
@export var voice_pitch: float = 1.0

# Optional: Next dialogue in chain
@export var next_dialogue: DialogueData

# dialogue_intro.tres:
# - character_name: "Village Elder"
# - lines: ["Welcome, traveler!", "We need your help."]

# dialogue_quest.tres:
# - character_name: "Village Elder"
# - lines: ["Goblins have been...", "Can you help us?"]
```

### 4. Level Configuration

```gd
# ============================================
# LEVEL RESOURCE
# ============================================
class_name LevelData extends Resource

@export_group("Level Info")
@export var level_name: String
@export var scene_path: String
@export var thumbnail: Texture2D

@export_group("Difficulty")
@export var recommended_level: int = 1
@export var enemy_health_multiplier: float = 1.0
@export var enemy_damage_multiplier: float = 1.0

@export_group("Spawn Settings")
@export var enemy_types: Array[EnemyData] = []
@export var max_enemies: int = 10
@export var spawn_interval: float = 5.0

@export_group("Rewards")
@export var completion_experience: int = 100
@export var completion_gold: int = 50

# level_1_forest.tres
# level_2_cave.tres
# level_3_castle.tres
```

### 5. Audio Settings

```gd
# ============================================
# AUDIO PRESET RESOURCE
# ============================================
class_name AudioPreset extends Resource

@export var preset_name: String
@export_range(0, 1) var master_volume: float = 1.0
@export_range(0, 1) var music_volume: float = 0.7
@export_range(0, 1) var sfx_volume: float = 1.0
@export_range(0, 1) var voice_volume: float = 1.0

func apply():
    AudioServer.set_bus_volume_db(0, linear_to_db(master_volume))
    AudioServer.set_bus_volume_db(1, linear_to_db(music_volume))
    AudioServer.set_bus_volume_db(2, linear_to_db(sfx_volume))

# audio_preset_default.tres
# audio_preset_quiet.tres
# audio_preset_loud.tres
```

### 6. Spell/Ability Data

```gd
# ============================================
# SPELL RESOURCE
# ============================================
class_name SpellData extends Resource

@export_group("Identity")
@export var spell_name: String
@export var icon: Texture2D
@export_multiline var description: String

@export_group("Mechanics")
@export var damage: int = 0
@export var healing: int = 0
@export var mana_cost: int = 10
@export var cooldown: float = 1.0
@export var cast_time: float = 0.0
@export var range: float = 100.0

@export_group("Visual")
@export var projectile_scene: PackedScene
@export var cast_animation: String = "cast"
@export var particle_effect: PackedScene

@export_group("Audio")
@export var cast_sound: AudioStream
@export var impact_sound: AudioStream

# fireball.tres
# heal.tres
# lightning_bolt.tres
```

## Advanced Techniques

### Resource Arrays

```gd
# ============================================
# COLLECTIONS OF RESOURCES
# ============================================
class_name WeaponDatabase extends Resource

@export var all_weapons: Array[WeaponData] = []

func get_weapon_by_name(name: String) -> WeaponData:
    for weapon in all_weapons:
        if weapon.weapon_name == name:
            return weapon
    return null

# weapon_database.tres contains array of all weapon .tres files
```

### Nested Resources

```gd
# ============================================
# RESOURCES CONTAINING RESOURCES
# ============================================
class_name CharacterStats extends Resource

@export var character_name: String
@export var starting_weapon: WeaponData  # Another .tres
@export var starting_items: Array[ItemData] = []  # Array of .tres

# warrior_stats.tres:
# - starting_weapon: iron_sword.tres
# - starting_items: [health_potion.tres, bread.tres]
```

### Resource Inheritance

```gd
# ============================================
# BASE RESOURCE
# ============================================
class_name ItemData extends Resource

@export var item_name: String
@export var icon: Texture2D
@export var value: int

# ============================================
# SPECIALIZED RESOURCE
# ============================================
class_name ConsumableItem extends ItemData

@export var health_restore: int = 0
@export var mana_restore: int = 0
@export var buff_duration: float = 0.0

# health_potion.tres (ConsumableItem)
# mana_potion.tres (ConsumableItem)
```

## Creating Resources in Code

```gd
# ============================================
# RUNTIME RESOURCE CREATION
# ============================================
func create_weapon() -> WeaponData:
    var weapon = WeaponData.new()
    weapon.weapon_name = "Legendary Sword"
    weapon.damage = 100
    weapon.attack_speed = 2.0
    return weapon

func save_custom_resource():
    var weapon = create_weapon()
    # Save to disk
    ResourceSaver.save(weapon, "user://custom_weapon.tres")

func load_custom_resource() -> WeaponData:
    return load("user://custom_weapon.tres") as WeaponData
```

## Resource vs RefCounted

| Feature | Resource (.tres) | RefCounted |
|---------|------------------|------------|
| Saveable to file | ✅ Yes | ❌ No |
| Editable in Inspector | ✅ Yes | ❌ No |
| Reusable instances | ✅ Yes (shared) | ⚠️ Must create new |
| Memory management | Auto (reference) | Auto (reference) |
| Best for | Data templates | Runtime logic |

```gd
# RESOURCE - Save and reuse
var sword = load("res://sword.tres")  # Same instance everywhere

# REFCOUNTED - Create new each time
var player_data = PlayerData.new()  # Unique instance
```

## Best Practices

```gd
# ============================================
# ORGANIZE BY FOLDERS
# ============================================
# res://data/
#   ├── weapons/
#   │   ├── sword.tres
#   │   ├── axe.tres
#   ├── enemies/
#   │   ├── goblin.tres
#   │   ├── orc.tres
#   ├── items/
#       ├── potions/
#       ├── materials/

# ============================================
# USE DESCRIPTIVE NAMES
# ============================================
# ✅ iron_sword.tres
# ✅ health_potion_small.tres
# ❌ item1.tres
# ❌ data.tres

# ============================================
# GROUP RELATED EXPORTS
# ============================================
@export_group("Combat")
@export var damage: int
@export var attack_speed: float

@export_group("Visual")
@export var icon: Texture2D
@export var sprite: Texture2D

# ============================================
# ADD HELPER FUNCTIONS
# ============================================
class_name WeaponData extends Resource

@export var damage: int
@export var attack_speed: float

# Calculated property
func get_dps() -> float:
    return damage * attack_speed

# Validation
func is_valid() -> bool:
    return damage > 0 and attack_speed > 0
```

## Common Pitfalls

```gd
# ============================================
# PITFALL 1: Modifying shared resources
# ============================================
# ❌ BAD: All enemies share same resource
@export var enemy_data: EnemyData

func take_damage(amount: int):
    enemy_data.max_health -= amount  # Modifies the .tres file!

# ✅ GOOD: Store instance data separately
@export var enemy_data: EnemyData
var current_health: int

func _ready():
    current_health = enemy_data.max_health  # Copy value

func take_damage(amount: int):
    current_health -= amount  # Modify instance, not resource

# ============================================
# PITFALL 2: Forgetting to save changes
# ============================================
# Changes in Inspector only save when you Ctrl+S or click save icon
```

__Quick Summary:__

- `.tres` = Reusable data templates (like structs/JSON)
- Extend `Resource` to create custom types
- Create `.tres` files in editor (Right-click → New Resource)
- Assign via `@export` or `load()`
- Great for items, enemies, spells, levels, configs
- Shared by default - copy values if you need instance data
