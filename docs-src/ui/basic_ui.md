
# Basic UI

## UI Scene

```gd
# ============================================
# UI NODE TYPES
# ============================================
# Control - Base UI node (like AS3 UIComponent)
# Container nodes (auto-layout):
#   - VBoxContainer (vertical stack)
#   - HBoxContainer (horizontal)
#   - GridContainer (grid layout)
#   - MarginContainer (padding)
# Widgets:
#   - Button, Label, LineEdit (text input)
#   - ProgressBar, TextureRect, Panel

# ============================================
# BASIC UI SCENE
# ============================================
extends Control

@onready var health_bar: ProgressBar = $HealthBar
@onready var gold_label: Label = $GoldLabel
@onready var inventory_grid: GridContainer = $InventoryGrid

func _ready():
    # Update UI:
    update_health(100, 100)
    update_gold(500)
    
    # Connect to game signals:
    PlayerManager.health_changed.connect(update_health)
    EconomyManager.gold_changed.connect(update_gold)

func update_health(current: int, max: int):
    health_bar.value = (float(current) / max) * 100

func update_gold(amount: int):
    gold_label.text = "Gold: %d" % amount
```


## Theming (like CSS)

- Create a Theme resource in editor
- Apply to Control node
- Customize fonts, colors, sizes globally

## Anchors & Offset Transforms

!!! tip "New in Godot 4.7 — Control offset transforms"
    `Control` nodes now have extra offset transform properties that let you nudge UI elements without affecting anchors or layout. Useful for juice/animation without fighting containers.

## Anchors

Set in Inspector:

- Anchor Preset -> Top Left (default)
- Anchor Preset -> Center (centers on screen)
- Anchor Preset -> Full Rect (stretches with window)
- Anchor Preset -> Top Wide (full width, anchored top)


## Signals

```gd
# ============================================
# SIGNALS
# ============================================
@onready var attack_btn: Button = $AttackButton

func _ready():
    attack_btn.pressed.connect(_on_attack_pressed)
    attack_btn.mouse_entered.connect(_on_attack_hover)

func _on_attack_pressed():
    print("Attack!")

func _on_attack_hover():
    attack_btn.modulate = Color.YELLOW
```
