# Visual Effects

## Particle Systems

Godot 4 ships with both CPU- and GPU-driven 2D particles. `CPUParticles2D` is the easiest to set up and works everywhere. `GPUParticles2D` can push more particles with less CPU cost, but it needs a `ParticleProcessMaterial` or a custom shader.

### Emitter Types

- `CPUParticles2D` — Runs on the CPU; full Inspector control.
- `GPUParticles2D` — Runs on the GPU; uses a `ParticleProcessMaterial`.
- `CPUParticles3D` / `GPUParticles3D` — The 3D equivalents.

### Common Parameters

- `amount` — Maximum active particles.
- `lifetime` — Seconds each particle lives.
- `one_shot` — Emit once and stop.
- `explosiveness` — `0` = steady stream, `1` = instant burst.
- `preprocess` — Simulate before becoming visible.
- `emission_shape` — Point, box, sphere, ring.
- `direction` / `spread` — Movement direction and random angle.
- `initial_velocity_min` / `initial_velocity_max`
- `gravity`
- `scale_amount_min` / `scale_amount_max`
- `color` / `color_ramp`
- `trail_enabled` — Draw a trail behind each particle.

### Emitter Behaviors

Looping, one-shot, and timed bursts are controlled by `emitting`, `one_shot`, and `preprocess`. `preprocess` lets a looping effect run for a few seconds before the player sees it, so it already looks in motion.

### Emitter Effects

Effects are combinations of the above. A "glow" effect is usually bright additive particles combined with a `WorldEnvironment` that has Glow enabled. A "directional" effect sends particles along a narrow cone. A "trail" effect sets `trail_enabled` and can use a long `lifetime` with low `initial_velocity`.

### Burst Example

```gd
# ============================================
# Burst using CPUParticles2D
# ============================================
extends CPUParticles2D

func burst():
    amount = 30
    lifetime = 0.4
    one_shot = true
    explosiveness = 1.0
    direction = Vector2.UP
    spread = 180.0
    initial_velocity_min = 80.0
    initial_velocity_max = 150.0
    emitting = true
```

### Glow Example

```gd
# ============================================
# Glowy particles with additive blend
# ============================================
extends CPUParticles2D

func _ready():
    amount = 80
    lifetime = 1.5
    color = Color.YELLOW
    color_ramp = load("res://glow_gradient.tres")
    scale_amount_min = 0.5
    scale_amount_max = 1.5
    emitting = true

    # Additive blend makes overlapping particles brighter
    var mat = CanvasItemMaterial.new()
    mat.blend_mode = CanvasItemMaterial.BLEND_MODE_ADD
    material = mat
```

__Tip:__ To make the glow bloom, add a `WorldEnvironment` with an `Environment` whose `background_mode` is `Canvas` and `glow_enabled` is `true`.

### Directional Example

```gd
# ============================================
# Directional stream (e.g. rocket exhaust)
# ============================================
extends CPUParticles2D

func _ready():
    amount = 100
    lifetime = 0.8
    emission_shape = EMISSION_SHAPE_POINT
    direction = Vector2.RIGHT
    spread = 15.0
    initial_velocity_min = 200.0
    initial_velocity_max = 300.0
    gravity = Vector2.ZERO
    emitting = true
```

### Trail Example

```gd
# ============================================
# Particle trail
# ============================================
extends CPUParticles2D

func _ready():
    amount = 30
    lifetime = 1.2
    trail_enabled = true
    trail_length = 0.4
    initial_velocity_min = 20.0
    initial_velocity_max = 40.0
    scale_amount_min = 0.2
    scale_amount_max = 0.5
    emitting = true
```

## Basic Lighting Systems

2D lights in Godot 4 are `Light2D` nodes. They only affect `CanvasItem` nodes on the same or compatible `light_mask` layers.

### Light Types

- `PointLight2D` — Omnidirectional light from a single point.
- `SpotLight2D` — Cone-shaped light, like a flashlight.
- `DirectionalLight2D` — Infinite parallel light, like the sun.
- `WorldEnvironment` — Sets the canvas background/ambient.

### Common Parameters

- `color` — Light tint.
- `energy` — Brightness multiplier.
- `texture` — Optional mask/texture for the light shape.
- `shadow_enabled` — Casts shadows from `LightOccluder2D` nodes.
- `range_item_cull_mask` / `shadow_item_cull_mask` — Which nodes the light and shadow affect.
- `blend_mode` — `BLEND_MODE_ADD`, `BLEND_MODE_MIX`, or `BLEND_MODE_SUB`.

### Light Behaviors

A `PointLight2D` illuminates a radius around itself. A `DirectionalLight2D` uses its `direction` property to cast angled light across the scene. A `SpotLight2D` uses `angle` and `direction` to form a cone. `WorldEnvironment` with a canvas background sets the ambient color.

### Ambient Light Example

```gd
# ============================================
# WorldEnvironment for 2D ambient light
# ============================================
extends WorldEnvironment

func _ready():
    environment = Environment.new()
    environment.background_mode = Environment.BG_CANVAS
    environment.ambient_light_source = Environment.AMBIENT_SOURCE_COLOR
    environment.ambient_light_color = Color(0.15, 0.15, 0.25)
    environment.ambient_light_energy = 0.6
```

### Directional Light Example

```gd
# ============================================
# 2D sun/moon light
# ============================================
extends DirectionalLight2D

func _ready():
    color = Color.ORANGE
    energy = 1.2
    direction = Vector2(-1, -0.5).normalized()
    shadow_enabled = true
```

### Point Light Example

```gd
# ============================================
# Lantern or torch
# ============================================
extends PointLight2D

func _ready():
    color = Color.ORANGE
    energy = 1.5
    texture = load("res://light_cookie.png")
    shadow_enabled = true
```

### Spot Light Example

```gd
# ============================================
# Flashlight
# ============================================
extends SpotLight2D

func _ready():
    color = Color.WHITE
    energy = 2.0
    direction = Vector2.RIGHT
    angle = 30.0  # cone width in degrees
    shadow_enabled = true
```
