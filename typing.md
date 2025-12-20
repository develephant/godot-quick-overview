
# Typing

Typing is optional. All of these work:

```php
# Fully typed (most explicit):
@onready var health_bar: ProgressBar = $HealthBar

# Inferred typing (Godot figures it out):
@onready var health_bar := $HealthBar

# Untyped (dynamic, like Lua):
@onready var health_bar = $HealthBar
```

The inferred sweet spot:

```php
# Clean AND safe:
@onready var health_bar := $HealthBar as ProgressBar

# The 'as' gives you:
# - Type safety (autocomplete works)
# - Runtime type check (fails gracefully if node is wrong type)
# - Less verbose than full typing
```
