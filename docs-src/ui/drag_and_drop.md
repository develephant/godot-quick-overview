# Drag and Drop

Drag and drop in Godot 4 UI uses three `Control` callbacks. Returning data from `_get_drag_data` starts the drag. The target `Control` accepts the drop with `_can_drop_data` and receives it with `_drop_data`.

## Basic Setup

The source `Control` needs:

- `_get_drag_data(at_position: Vector2) -> Variant`
- Set a preview with `set_drag_preview(control)`.

The target `Control` needs:

- `_can_drop_data(at_position: Vector2, data: Variant) -> bool`
- `_drop_data(at_position: Vector2, data: Variant)`

## Data Transfer

The data returned by `_get_drag_data` can be any `Variant` — a `String`, a `Dictionary`, a `Resource`, etc. The same value is passed to `_can_drop_data` and `_drop_data` on the target.

## Visual Feedback

Call `set_drag_preview(preview: Control)` inside `_get_drag_data`. The preview is a `Control` (often a `TextureRect` or `Label`) that follows the cursor. Returning `null` cancels the drag.

## Drop Targets

A target validates the payload in `_can_drop_data` and applies it in `_drop_data`. Returning `false` in `_can_drop_data` shows the no-drop cursor.

## Example: Simple drag and drop

### Source

```gd
# ============================================
# Color source
# ============================================
extends ColorRect

func _get_drag_data(_at_position: Vector2):
    var preview = ColorRect.new()
    preview.size = Vector2(32, 32)
    preview.color = color
    set_drag_preview(preview)
    return color
```

### Target

```gd
# ============================================
# Color target
# ============================================
extends ColorRect

func _can_drop_data(_at_position: Vector2, data) -> bool:
    return data is Color

func _drop_data(_at_position: Vector2, data):
    if data is Color:
        color = data
```

## Example: Complex drag and drop

```gd
# ============================================
# Inventory slot
# ============================================
extends Button

@export var slot_index: int = 0
var item_data: Dictionary = {}

func _get_drag_data(_at_position: Vector2):
    if item_data.is_empty():
        return null

    # Build a preview of the item
    var preview = Label.new()
    preview.text = item_data.get("name", "Item")
    preview.add_theme_color_override("font_color", Color.WHITE)
    set_drag_preview(preview)

    # Include the origin so the receiver knows where it came from
    return {
        "from_slot": slot_index,
        "item": item_data
    }

func _can_drop_data(_at_position: Vector2, data) -> bool:
    if not data is Dictionary:
        return false
    if not data.has("item"):
        return false
    # Don't drop on the same slot
    return data.get("from_slot") != slot_index

func _drop_data(_at_position: Vector2, data):
    if not data is Dictionary:
        return

    var source_slot = data.get("from_slot")

    # Swap logic lives in the inventory manager
    InventoryManager.swap_items(source_slot, slot_index)
```
