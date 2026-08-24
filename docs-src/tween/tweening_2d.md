# Tweening (2D)

## Basic Tween Creation

```gd
# ============================================
# CREATING TWEENS
# ============================================
extends Node2D

func _ready():
    # Create a new tween
    var tween = create_tween()
    
    # Tween position over 2 seconds
    tween.tween_property($Sprite2D, "position", Vector2(500, 300), 2.0)
```

## Common Tween Properties

```gd
# ============================================
# POSITION
# ============================================
func move_to_position():
    var tween = create_tween()
    tween.tween_property($Sprite2D, "position", Vector2(500, 300), 1.5)

# ============================================
# SCALE
# ============================================
func scale_up():
    var tween = create_tween()
    tween.tween_property($Sprite2D, "scale", Vector2(2, 2), 1.0)

# ============================================
# ROTATION
# ============================================
func rotate_object():
    var tween = create_tween()
    # Rotation in radians
    tween.tween_property($Sprite2D, "rotation", TAU, 2.0)  # 360 degrees

# ============================================
# MODULATE (Color/Transparency)
# ============================================
func fade_out():
    var tween = create_tween()
    tween.tween_property($Sprite2D, "modulate:a", 0.0, 1.0)  # Alpha to 0

func change_color():
    var tween = create_tween()
    tween.tween_property($Sprite2D, "modulate", Color.RED, 0.5)
```

## Easing & Transitions

```gd
# ============================================
# EASING FUNCTIONS
# ============================================
func smooth_movement():
    var tween = create_tween()
    tween.set_ease(Tween.EASE_IN_OUT)  # Start slow, end slow
    tween.set_trans(Tween.TRANS_CUBIC)
    tween.tween_property($Sprite2D, "position", Vector2(500, 300), 1.0)

# Common easing types:
# EASE_IN - Slow start
# EASE_OUT - Slow end
# EASE_IN_OUT - Slow start and end
# EASE_OUT_IN - Fast start and end

# Common transitions:
# TRANS_LINEAR - Constant speed
# TRANS_SINE - Smooth curve
# TRANS_CUBIC - Sharper curve
# TRANS_ELASTIC - Bouncy overshoot
# TRANS_BOUNCE - Bounces at the end
# TRANS_BACK - Slight overshoot
```

## Chaining Tweens

```gd
# ============================================
# SEQUENTIAL TWEENS
# ============================================
func sequence_animation():
    var tween = create_tween()
    
    # Move right
    tween.tween_property($Sprite2D, "position:x", 500, 1.0)
    # Then move down
    tween.tween_property($Sprite2D, "position:y", 400, 1.0)
    # Then fade out
    tween.tween_property($Sprite2D, "modulate:a", 0.0, 0.5)

# ============================================
# PARALLEL TWEENS
# ============================================
func parallel_animation():
    var tween = create_tween()
    
    # All happen at the same time
    tween.set_parallel(true)
    tween.tween_property($Sprite2D, "position", Vector2(500, 300), 1.0)
    tween.tween_property($Sprite2D, "scale", Vector2(2, 2), 1.0)
    tween.tween_property($Sprite2D, "rotation", TAU, 1.0)

# ============================================
# MIXED SEQUENTIAL AND PARALLEL
# ============================================
func complex_animation():
    var tween = create_tween()
    
    # First move (sequential)
    tween.tween_property($Sprite2D, "position", Vector2(500, 300), 1.0)
    
    # Then scale and rotate together (parallel)
    tween.set_parallel(true)
    tween.tween_property($Sprite2D, "scale", Vector2(2, 2), 0.5)
    tween.tween_property($Sprite2D, "rotation", TAU, 0.5)
    
    # Back to sequential
    tween.set_parallel(false)
    tween.tween_property($Sprite2D, "modulate:a", 0.0, 0.5)
```

## Tween Control

```gd
# ============================================
# DELAYS & CALLBACKS
# ============================================
func animation_with_delay():
    var tween = create_tween()
    
    tween.tween_property($Sprite2D, "position:x", 500, 1.0)
    tween.tween_interval(0.5)  # Wait 0.5 seconds
    tween.tween_property($Sprite2D, "position:y", 400, 1.0)
    tween.tween_callback(on_animation_complete)  # Call function when done

func on_animation_complete():
    print("Animation finished!")

# ============================================
# LOOPING
# ============================================
func loop_animation():
    var tween = create_tween()
    tween.set_loops()  # Infinite loop
    tween.tween_property($Sprite2D, "position:x", 500, 1.0)
    tween.tween_property($Sprite2D, "position:x", 100, 1.0)

func loop_limited():
    var tween = create_tween()
    tween.set_loops(3)  # Loop 3 times
    tween.tween_property($Sprite2D, "scale", Vector2(1.5, 1.5), 0.5)
    tween.tween_property($Sprite2D, "scale", Vector2(1, 1), 0.5)

# ============================================
# PAUSING & CONTROLLING
# ============================================
var my_tween: Tween

func start_animation():
    my_tween = create_tween()
    my_tween.tween_property($Sprite2D, "position", Vector2(500, 300), 3.0)

func pause_animation():
    if my_tween:
        my_tween.pause()

func resume_animation():
    if my_tween:
        my_tween.play()

func stop_animation():
    if my_tween:
        my_tween.kill()
```

## Practical Examples

```gd
# ============================================
# BUTTON HOVER EFFECT
# ============================================
extends Button

func _ready():
    mouse_entered.connect(_on_mouse_entered)
    mouse_exited.connect(_on_mouse_exited)

func _on_mouse_entered():
    var tween = create_tween()
    tween.set_ease(Tween.EASE_OUT)
    tween.set_trans(Tween.TRANS_ELASTIC)
    tween.tween_property(self, "scale", Vector2(1.1, 1.1), 0.3)

func _on_mouse_exited():
    var tween = create_tween()
    tween.tween_property(self, "scale", Vector2(1, 1), 0.2)
```

```gd
# ============================================
# COIN COLLECT ANIMATION
# ============================================
extends Sprite2D

func collect():
    var tween = create_tween()
    tween.set_parallel(true)
    
    # Move up and fade out
    tween.tween_property(self, "position:y", position.y - 50, 0.5)
    tween.tween_property(self, "modulate:a", 0.0, 0.5)
    
    tween.set_parallel(false)
    tween.tween_callback(queue_free)  # Delete when done
```

```gd
# ============================================
# DAMAGE FLASH
# ============================================
extends Sprite2D

func take_damage():
    var tween = create_tween()
    tween.tween_property(self, "modulate", Color.RED, 0.1)
    tween.tween_property(self, "modulate", Color.WHITE, 0.1)
```

```gd
# ============================================
# SHAKE EFFECT
# ============================================
extends Node2D

func shake(duration: float = 0.3, intensity: float = 10.0):
    var original_pos = position
    var tween = create_tween()
    
    var shake_count = int(duration / 0.05)
    tween.set_parallel(true)
    
    for i in shake_count:
        var offset = Vector2(
            randf_range(-intensity, intensity),
            randf_range(-intensity, intensity)
        )
        tween.tween_property(self, "position", original_pos + offset, 0.05)
        tween.tween_interval(0.05 * i)
    
    tween.set_parallel(false)
    tween.tween_property(self, "position", original_pos, 0.05)
```

```gd
# ============================================
# POPUP WINDOW
# ============================================
extends Control

func popup():
    # Start small and transparent
    scale = Vector2(0, 0)
    modulate.a = 0
    visible = true
    
    var tween = create_tween()
    tween.set_ease(Tween.EASE_OUT)
    tween.set_trans(Tween.TRANS_BACK)  # Slight overshoot
    tween.set_parallel(true)
    
    tween.tween_property(self, "scale", Vector2(1, 1), 0.3)
    tween.tween_property(self, "modulate:a", 1.0, 0.3)
```

## Tween Tips & 4.4+ Helpers

```gd
# ============================================
# START FROM THE CURRENT VALUE INSTEAD OF ZERO
# ============================================
func pulse_scale():
    var tween = create_tween()
    # from_current lets you animate relative to where the object already is
    tween.tween_property($Sprite2D, "scale", Vector2(1.5, 1.5), 0.2).from_current()
    tween.tween_property($Sprite2D, "scale", Vector2(1, 1), 0.2).from_current()

# ============================================
# SET A GLOBAL SPEED SCALE (4.4+)
# ============================================
func slow_motion_tween():
    var tween = create_tween()
    tween.set_speed_scale(0.5)  # Plays at half speed
    tween.tween_property($Sprite2D, "position", Vector2(500, 300), 1.0)
```

!!! tip "New in 4.4+"
    - `from()` and `from_current()` let you define the starting value explicitly or use the current one.
    - `set_speed_scale()` controls playback speed without changing durations.

## Best Practices

```gd
# ============================================
# CLEANUP OLD TWEENS
# ============================================
var current_tween: Tween

func animate():
    # Kill old tween if exists
    if current_tween:
        current_tween.kill()
    
    current_tween = create_tween()
    current_tween.tween_property($Sprite2D, "position", Vector2(500, 300), 1.0)
```

```gd
# ============================================
# CUSTOM TWEEN METHOD
# ============================================
func smooth_move(target_pos: Vector2, duration: float = 1.0):
    var tween = create_tween()
    tween.set_ease(Tween.EASE_IN_OUT)
    tween.set_trans(Tween.TRANS_CUBIC)
    tween.tween_property(self, "position", target_pos, duration)
    return tween
```

__Quick Reference:__

- **create_tween()** - Start a new tween
- **tween_property()** - Animate a property
- **tween_interval()** - Add delay
- **tween_callback()** - Call function
- **set_parallel(true/false)** - Toggle parallel mode
- **set_loops(count)** - Repeat animation
- **set_ease()** - Set easing style
- **set_trans()** - Set transition type
