# Signals

## Basic Signal Declaration & Emission

```gd
# ============================================
# DECLARING SIGNALS
# ============================================
extends Node

# Simple signal
signal health_changed

# Signal with parameters
signal player_died(player_name: String)
signal damage_taken(amount: int, damage_type: String)

func take_damage(amount: int):
    health -= amount
    health_changed.emit()
    
    if health <= 0:
        player_died.emit(player_name)
```

## Connecting Signals

```gd
# ============================================
# METHOD 1: Code Connection
# ============================================
extends Node

func _ready():
    # Connect to another node's signal
    $Player.health_changed.connect(_on_player_health_changed)
    $Player.player_died.connect(_on_player_died)

func _on_player_health_changed():
    print("Player health changed!")

func _on_player_died(player_name: String):
    print(player_name + " has died!")
```

```gd
# ============================================
# METHOD 2: Editor Connection (Inspector)
# ============================================
# 1. Select the node with the signal in Scene tree
# 2. Go to Node tab (next to Inspector)
# 3. Double-click signal → pick receiver node → select/create method
# 4. Godot auto-generates method with correct signature
```

```gd
# ============================================
# METHOD 3: Lambda/Anonymous Functions
# ============================================
func _ready():
    $Button.pressed.connect(func(): print("Button pressed!"))
    
    # With parameters
    $Player.damage_taken.connect(func(amount, type): 
        print("Took " + str(amount) + " " + type + " damage")
    )
```

## Advanced Signal Patterns

```gd
# ============================================
# DISCONNECTING SIGNALS
# ============================================
func _ready():
    $Player.health_changed.connect(_on_health_changed)

func cleanup():
    if $Player.health_changed.is_connected(_on_health_changed):
        $Player.health_changed.disconnect(_on_health_changed)
```

```gd
# ============================================
# ONE-SHOT SIGNALS (Connect once, auto-disconnect)
# ============================================
func _ready():
    # CONNECT_ONE_SHOT flag
    $Timer.timeout.connect(_on_timer_timeout, CONNECT_ONE_SHOT)

func _on_timer_timeout():
    print("This only fires once!")
```

```gd
# ============================================
# SIGNAL CHAINING
# ============================================
extends Node

signal game_started
signal wave_started(wave_number: int)

func _ready():
    game_started.connect(_on_game_started)

func _on_game_started():
    # One signal triggers another
    wave_started.emit(1)
```

## Global Signal Bus Pattern

```gd
# ============================================
# AUTOLOAD: SignalBus.gd
# ============================================
extends Node

# UI signals
signal score_changed(new_score: int)
signal show_notification(message: String)

# Game state signals
signal level_completed(level_id: int)
signal enemy_spawned(enemy: Node)

# This is accessible from anywhere as SignalBus.signal_name
```

```gd
# ============================================
# USING SIGNAL BUS - Any Script
# ============================================
extends Node

func _ready():
    # Connect to global signals
    SignalBus.score_changed.connect(_on_score_changed)
    SignalBus.level_completed.connect(_on_level_completed)

func collect_coin():
    # Emit global signal from anywhere
    SignalBus.score_changed.emit(current_score)
    SignalBus.show_notification.emit("Coin collected!")

func _on_score_changed(new_score: int):
    print("New score: " + str(new_score))
```

## Awaiting Signals

```gd
# ============================================
# WAIT FOR A SIGNAL BEFORE CONTINUING
# ============================================
func play_dialogue():
    show_text_box("Hello!")
    await $TextBox.finished
    print("Dialogue done, do something next")

func hurt_player():
    $Player.flash()
    await get_tree().create_timer(0.2).timeout
    $Player.take_damage(10)
```

!!! tip "Await works with any signal or Callable"
    `await some_signal` pauses the function until the signal is emitted. Use `await some_node.some_callable` to wait for a one-shot `Callable`.

## Built-in Node Signals

```gd
# ============================================
# COMMON BUILT-IN SIGNALS
# ============================================

# Node signals
ready  # When node enters scene tree
tree_entered
tree_exiting

# Button signals
pressed
toggled(toggled_on: bool)

# Timer signals
timeout

# AnimationPlayer signals
animation_finished(anim_name: String)

# Area2D/Area3D signals
body_entered(body: Node2D)
body_exited(body: Node2D)
area_entered(area: Area2D)

# Example usage
func _ready():
    $Button.pressed.connect(_on_button_pressed)
    $Timer.timeout.connect(_on_timer_timeout)
    $AnimationPlayer.animation_finished.connect(_on_animation_finished)
```

## Best Practices

```gd
# ============================================
# NAMING CONVENTIONS
# ============================================

# Signal names: past tense or state change
signal died
signal health_changed
signal item_collected

# Handler methods: _on_source_signal_name
func _on_player_died():
    pass

func _on_enemy_health_changed():
    pass
```

```gd
# ============================================
# CLEANUP
# ============================================
extends Node

var connected_signals = []

func _ready():
    $Player.died.connect(_on_player_died)

func _exit_tree():
    # Disconnect all signals when node is removed
    if $Player and $Player.died.is_connected(_on_player_died):
        $Player.died.disconnect(_on_player_died)
```

__When to use what:__

- **Custom Signals** - Communication between specific game objects
- **Signal Bus** - Global events (UI updates, game state changes)
- **Built-in Signals** - Responding to engine events (collisions, animations, input)
- **Code Connection** - Dynamic connections, flexibility needed
- **Editor Connection** - Simple static connections, easier debugging
