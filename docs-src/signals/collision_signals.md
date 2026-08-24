
# Collision Detection with Signals

## Trigger Volume (no physics)

```gd
# ============================================
# AREA2D - Trigger volumes (no physics)
# ============================================
extends Area2D

signal player_entered
signal player_exited
signal coin_collected(value: int)

func _ready():
    # Connect built-in signals:
    body_entered.connect(_on_body_entered)
    body_exited.connect(_on_body_exited)
    area_entered.connect(_on_area_entered)

func _on_body_entered(body: Node2D):
    if body.is_in_group("player"):
        player_entered.emit()
        print("Player entered trigger!")

func _on_area_entered(area: Area2D):
    if area.is_in_group("coins"):
        coin_collected.emit(area.value)
        area.queue_free()
```

## Physics Collision

```gd
# ============================================
# CHARACTERBODY2D - Physics collisions
# ============================================
extends CharacterBody2D

signal hit_enemy(enemy: Node)
signal hit_wall

func _physics_process(delta):
    move_and_slide()
    
    # Check what we collided with:
    for i in get_slide_collision_count():
        var collision = get_slide_collision(i)
        var collider = collision.get_collider()
        
        if collider.is_in_group("enemies"):
            hit_enemy.emit(collider)
        elif collider is TileMap or collider is TileMapLayer:
            hit_wall.emit()
```

## Raycast (line-of-sight)

```gd
# ============================================
# RAYCAST2D - Line-of-sight checks
# ============================================
extends RayCast2D

signal player_spotted

func _physics_process(delta):
    if is_colliding():
        var collider = get_collider()
        if collider.is_in_group("player"):
            player_spotted.emit()
```

__Example:__

```gd
# ============================================
# PRACTICAL EXAMPLE - Damage zone
# ============================================
extends Area2D

@export var damage_per_second: float = 10.0

var bodies_in_zone: Array[CharacterBody2D] = []

func _ready():
    body_entered.connect(_on_body_entered)
    body_exited.connect(_on_body_exited)

func _on_body_entered(body):
    if body.has_method("take_damage"):
        bodies_in_zone.append(body)

func _on_body_exited(body):
    bodies_in_zone.erase(body)

func _process(delta):
    for body in bodies_in_zone:
        body.take_damage(damage_per_second * delta)

```
