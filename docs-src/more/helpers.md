
# Helpers

## `self`

A reference in code to the current `Object` instance.

```gd
var my_name = self.name
```

__NOTE__: `self` will only work in `Object` instances


## `$` Expansion (-> get_node)

```gd
var item = $SomeNode

# same as get_tree().get_node("SomeNode")
```

## `await`

Wait on a process.

```gd
await get_tree().create_timer(1.5).timeout
# waiting 1.5 seconds...
print("timer done!")
```

## String Replace

```gd
var str = "You got %d coins" % 10
```

- `%d` - Number token
- `%s` - String token


## Auto-Type

```gd
var item := "Boxes"
```

## Node Class

Providing a class name will automatically register the class as global. 

It will also be available in the `Node` selector.

```gd
class_name CargoBox extends Node2D

func _init():
    print("create cargo")

# USAGE (anywhere):
var cargo = CargoBox.new()
```

## RefCounted Class

This creates a auto-registered `RefCounted` static class called `Food`.

```gd
class_name Food extends RefCounted

static func make_tacos(amount: int) -> String:
    var response = "You made %d tacos!" % amount
    print(response)
    return response
```

__USAGE:__

```gd
# Access from anywhere

var tacos_made = Food.make_tacos(3)
```
