
# Class Instances

## Data Class Instance

```gd
# ============================================
# DATA CLASS INSTANCES
# ============================================
var player_data = PlayerData.new("Chris", 5)
var another_data := PlayerData.new()  # Uses defaults
```

## Node Instances (programmatic)

```gd
# ============================================
# NODE INSTANCES (programmatic)
# ============================================
# Load scene file:
var PlayerScene = load("res://player.tscn")
# Or preload (compile-time, faster):
const PlayerScene = preload("res://player.tscn")

# Create instance:
var player = PlayerScene.instantiate()

# Configure before adding to tree:
player.position = Vector2(100, 100)
player.player_data = PlayerData.new("Bob", 1)

# Add to tree (now it exists in game):
add_child(player)
```

## Node Instances

```gd
# ============================================
# NODE INSTANCES (from code-only class)
# ============================================
var sprite = Sprite2D.new()
sprite.texture = load("res://icon.png")
sprite.position = Vector2(200, 200)
add_child(sprite)
```


## Resource Instances

```gd
# ============================================
# RESOURCE INSTANCES
# ============================================
# Load from file:
var sword = load("res://items/sword.tres") as WeaponData
print(sword.damage)

# Or create in code:
var bow = WeaponData.new()
bow.weapon_name = "Shortbow"
bow.damage = 15
```


