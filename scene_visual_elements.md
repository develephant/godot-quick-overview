
#  Instantiating and Placing Visual Elements

## Basic

```php
extends Node2D

const EnemyScene = preload("res://enemy.tscn")
const CoinScene = preload("res://coin.tscn")

# ============================================
# BASIC SPAWNING
# ============================================
func spawn_enemy(at_position: Vector2):
    var enemy = EnemyScene.instantiate()
    enemy.position = at_position
    add_child(enemy)
    return enemy
```

## Spawn with Config

```php
# ============================================
# SPAWNING WITH CONFIGURATION
# ============================================
func spawn_enemy_advanced(at_position: Vector2, level: int):
    var enemy = EnemyScene.instantiate()
    
    # Configure BEFORE adding to tree:
    enemy.position = at_position
    enemy.set_level(level)
    enemy.add_to_group("enemies")
    
    # Add to tree:
    add_child(enemy)
    
    # Connect signals:
    enemy.died.connect(_on_enemy_died)
    
    return enemy
```

## Spawn in Parent

```php
# ============================================
# SPAWNING IN SPECIFIC PARENT
# ============================================
@onready var enemy_container: Node2D = $EnemyContainer

func spawn_organized():
    var enemy = EnemyScene.instantiate()
    enemy.position = Vector2(100, 100)
    enemy_container.add_child(enemy)  # Goes under specific node
```

## Spawn Dynamic UI

```php
# ============================================
# DYNAMIC UI SPAWNING
# ============================================
const InventorySlotScene = preload("res://ui/inventory_slot.tscn")

@onready var inventory_grid: GridContainer = $UI/InventoryGrid

func populate_inventory(items: Array):
    # Clear existing:
    for child in inventory_grid.get_children():
        child.queue_free()
    
    # Spawn new:
    for item in items:
        var slot = InventorySlotScene.instantiate()
        slot.set_item(item)
        inventory_grid.add_child(slot)
```

## Pooling

```php
# ============================================
# POOLING PATTERN (for performance)
# ============================================
var bullet_pool: Array[Node2D] = []
const BulletScene = preload("res://bullet.tscn")

func get_bullet() -> Node2D:
    # Reuse inactive bullet if available:
    for bullet in bullet_pool:
        if not bullet.visible:
            bullet.visible = true
            return bullet
    
    # Create new if pool empty:
    var bullet = BulletScene.instantiate()
    add_child(bullet)
    bullet_pool.append(bullet)
    return bullet

func return_bullet(bullet: Node2D):
    bullet.visible = false
    bullet.position = Vector2(-1000, -1000)  # Off-screen
```
