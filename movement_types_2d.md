# Movement Types in Godot 4 (2D)

## Overview

Not all moveable objects need physics! Godot offers multiple movement approaches depending on your needs.

## 1. Direct Transform Movement (Non-Physics)

```php
# ============================================
# SIMPLE POSITION MOVEMENT
# ============================================
extends Node2D

@export var speed: float = 100.0

func _process(delta):
    # Direct position change
    position.x += speed * delta
    position.y += speed * delta
    
    # Or use += with Vector2
    position += Vector2(speed, 0) * delta

func _process(delta):
    # Rotation
    rotation += 1.0 * delta
    
    # Scale
    scale += Vector2(0.1, 0.1) * delta
```

**Characteristics:**
- No collision detection by default
- Fastest performance
- Full manual control
- Objects pass through each other

**Use Cases:**
- Background parallax layers
- Decorative objects
- Visual effects (particles, trails)
- UI elements
- Cutscene objects
- Non-interactive animations

```php
# ============================================
# EXAMPLE: FLOATING COIN
# ============================================
extends Sprite2D

var float_height: float = 10.0
var float_speed: float = 2.0
var time: float = 0.0

func _process(delta):
    time += delta
    # Sine wave movement
    position.y += sin(time * float_speed) * float_height * delta
```

## 2. CharacterBody2D (Kinematic Physics)

```php
# ============================================
# CONTROLLED MOVEMENT WITH COLLISIONS
# ============================================
extends CharacterBody2D

@export var speed: float = 200.0
@export var jump_velocity: float = -400.0
var gravity = ProjectSettings.get_setting("physics/2d/default_gravity")

func _physics_process(delta):
    # Apply gravity
    if not is_on_floor():
        velocity.y += gravity * delta
    
    # Get input
    var direction = Input.get_axis("left", "right")
    velocity.x = direction * speed
    
    # Jump
    if Input.is_action_just_pressed("jump") and is_on_floor():
        velocity.y = jump_velocity
    
    # Move with collision detection
    move_and_slide()
```

**Characteristics:**
- You control movement directly
- Built-in collision detection and response
- Helper functions: `is_on_floor()`, `is_on_wall()`
- Doesn't respond to external physics forces
- Predictable, responsive movement

**Use Cases:**
- Player characters
- NPCs with scripted movement
- Enemies
- Any character needing precise control
- Platformer characters
- Top-down RPG characters

```php
# ============================================
# EXAMPLE: TOP-DOWN MOVEMENT
# ============================================
extends CharacterBody2D

@export var speed: float = 300.0

func _physics_process(delta):
    # Get 2D movement input
    var input_vector = Input.get_vector("left", "right", "up", "down")
    
    # Set velocity
    velocity = input_vector * speed
    
    # Move with collisions
    move_and_slide()
```

## 3. RigidBody2D (Dynamic Physics)

```php
# ============================================
# PHYSICS-SIMULATED MOVEMENT
# ============================================
extends RigidBody2D

@export var thrust_force: float = 500.0

func _physics_process(delta):
    # Apply forces (physics engine handles movement)
    if Input.is_action_pressed("thrust"):
        apply_force(Vector2(thrust_force, 0))

func _ready():
    # Apply initial impulse
    apply_impulse(Vector2(100, -200))

func hit_by_explosion(explosion_position: Vector2, force: float):
    # Calculate direction from explosion
    var direction = global_position - explosion_position
    apply_impulse(direction.normalized() * force)
```

**Characteristics:**
- Physics engine controls movement
- Responds to gravity, forces, impulses
- Realistic physics behavior
- Can bounce, rotate, interact with other bodies
- Less predictable than CharacterBody2D

**Use Cases:**
- Projectiles (arrows, bullets)
- Physics objects (boxes, barrels)
- Ragdolls
- Destructible objects
- Physics puzzles
- Vehicles with realistic physics
- Bouncing objects

```php
# ============================================
# EXAMPLE: PHYSICS PROJECTILE
# ============================================
extends RigidBody2D

@export var launch_speed: float = 500.0

func launch(direction: Vector2):
    linear_velocity = direction.normalized() * launch_speed

func _on_body_entered(body):
    # Explode on impact
    explode()
    queue_free()
```

## 4. AnimatableBody2D (Animated Kinematic)

```php
# ============================================
# ANIMATION-DRIVEN MOVEMENT
# ============================================
extends AnimatableBody2D

# Moved by AnimationPlayer or Tween
# Other physics bodies react to its movement

func _ready():
    # Move with tween
    var tween = create_tween()
    tween.set_loops()
    tween.tween_property(self, "position:x", 500, 2.0)
    tween.tween_property(self, "position:x", 100, 2.0)
```

**Characteristics:**
- Moved by animations, tweens, or scripts
- Other physics bodies can ride on it
- Pushes CharacterBody2D and RigidBody2D
- Doesn't respond to physics forces
- Synchronized collision updates

**Use Cases:**
- Moving platforms
- Elevators
- Doors
- Conveyor belts
- Rotating platforms
- Crushing traps
- Any moving obstacle

```php
# ============================================
# EXAMPLE: MOVING PLATFORM
# ============================================
extends AnimatableBody2D

@export var move_distance: float = 300.0
@export var move_duration: float = 3.0

func _ready():
    var start_pos = position
    var end_pos = position + Vector2(move_distance, 0)
    
    var tween = create_tween()
    tween.set_loops()
    tween.set_trans(Tween.TRANS_SINE)
    tween.set_ease(Tween.EASE_IN_OUT)
    
    tween.tween_property(self, "position", end_pos, move_duration)
    tween.tween_property(self, "position", start_pos, move_duration)
```

## 5. StaticBody2D (No Movement)

```php
# ============================================
# STATIC COLLISION OBJECTS
# ============================================
extends StaticBody2D

# Used for walls, floors, obstacles
# Doesn't move, but can be moved manually in code
# Other bodies collide with it
```

**Characteristics:**
- Designed to never move
- Optimized for static collision
- Can be moved in code but not recommended for frequent movement

**Use Cases:**
- Walls
- Floors
- Static obstacles
- Invisible barriers

## 6. Area2D (Detection Only)

```php
# ============================================
# OVERLAP DETECTION (No Physics)
# ============================================
extends Area2D

signal player_entered

func _ready():
    body_entered.connect(_on_body_entered)

func _on_body_entered(body):
    if body.is_in_group("player"):
        player_entered.emit()

# Area2D can move freely
func _process(delta):
    position += Vector2(100, 0) * delta
```

**Characteristics:**
- Detects overlaps, no collision response
- Lightweight
- Can move freely
- No physics forces

**Use Cases:**
- Triggers
- Collectibles (coins, powerups)
- Detection zones (enemy sight range)
- Damage areas
- Level boundaries

## Movement Comparison

| Type | Collision | Physics Forces | Control | Performance | Best For |
|------|-----------|----------------|---------|-------------|----------|
| **Node2D** | ❌ No | ❌ No | ✅ Full | ⚡ Fastest | Visuals, UI |
| **CharacterBody2D** | ✅ Yes | ❌ No | ✅ Full | ⚡ Fast | Player, NPCs |
| **RigidBody2D** | ✅ Yes | ✅ Yes | ⚠️ Indirect | 🐌 Slower | Physics objects |
| **AnimatableBody2D** | ✅ Yes* | ❌ No | ✅ Full | ⚡ Fast | Platforms |
| **StaticBody2D** | ✅ Yes | ❌ No | ❌ None | ⚡ Fastest | Walls |
| **Area2D** | ⚠️ Detection | ❌ No | ✅ Full | ⚡ Fast | Triggers |

*Affects other bodies but not itself

## Decision Tree

```
Does it need to detect collisions?
├─ NO → Node2D (position movement)
└─ YES
   ├─ Does it need to respond to physics forces?
   │  ├─ YES → RigidBody2D
   │  └─ NO
   │     ├─ Does it move?
   │     │  ├─ NO → StaticBody2D
   │     │  └─ YES
   │     │     ├─ Do other objects need to ride on it?
   │     │     │  ├─ YES → AnimatableBody2D
   │     │     │  └─ NO → CharacterBody2D
   │     └─ Only detection needed?
   │        └─ YES → Area2D
```

## Hybrid Approaches

```php
# ============================================
# COMBINING MOVEMENT TYPES
# ============================================

# Example 1: Manual collision detection with Node2D
extends Node2D

var velocity: Vector2 = Vector2.ZERO

func _process(delta):
    position += velocity * delta
    check_manual_collisions()

func check_manual_collisions():
    # Use raycasts or Area2D for detection
    pass

# Example 2: CharacterBody2D with physics forces
extends CharacterBody2D

func _physics_process(delta):
    # Apply gravity manually
    velocity.y += 980 * delta
    
    # Apply wind force
    velocity.x += wind_force * delta
    
    move_and_slide()

# Example 3: Switch between RigidBody and CharacterBody
# Use RigidBody for ragdoll, then switch to CharacterBody when recovered
```

## Best Practices

```php
# ============================================
# CHOOSING THE RIGHT TYPE
# ============================================

# ✅ DO: Use CharacterBody2D for player
extends CharacterBody2D  # Responsive, predictable

# ❌ DON'T: Use RigidBody2D for player
extends RigidBody2D  # Slippery, hard to control

# ✅ DO: Use RigidBody2D for physics objects
extends RigidBody2D  # Realistic interactions

# ✅ DO: Use Node2D for background elements
extends Node2D  # No collision overhead

# ✅ DO: Use Area2D for collectibles
extends Area2D  # Lightweight detection
```

__Quick Reference:__

- **Visual only** → Node2D
- **Player control** → CharacterBody2D
- **Realistic physics** → RigidBody2D
- **Moving platforms** → AnimatableBody2D
- **Walls/floors** → StaticBody2D
- **Triggers/collectibles** → Area2D
