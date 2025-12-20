
# Node vs RefCounted

You can access the tree from `RefCounted`:

```php
# ============================================
# RefCounted CAN access tree - just needs a reference
# ============================================
class_name EconomyManager extends RefCounted

# This works fine!
func notify_all_shops():
    var tree = Engine.get_main_loop() as SceneTree
    var shops = tree.get_nodes_in_group("shops")
    for shop in shops:
        shop.update_prices()
```

OR if it's an autoload singleton:

```php
# ============================================
# RefCounted autoload - access via other singletons
# ============================================
class_name EconomyManager extends RefCounted

func notify_all_shops():
    # GameManager is a Node-based autoload
    var shops = GameManager.get_tree().get_nodes_in_group("shops")
    for shop in shops:
        shop.update_prices()
```

## When to use Node

You need to extend `Node` when you need to BE in the tree for these reasons:

1. _Lifecycle callbacks that require tree membership:_

    ```php
    # These ONLY work if you're a Node in the tree:
    extends Node

    func _ready():  # Called when added to tree
        pass

    func _process(delta):  # Called every frame
        pass

    func _physics_process(delta):  # Called every physics frame
        pass

    func _notification(what):  # Tree notifications
        if what == NOTIFICATION_PREDELETE:
            cleanup()
    ```

    __RefCounted alternative:__

    ```php
    # RefCounted - no automatic callbacks
    extends RefCounted

    func initialize():  # You call this manually
        pass

    # No _process, no _physics_process
    # If you need frame updates, something else has to call you
    ```

2. You need parent/child relationships:

    ```php
    extends Node

    func _ready():
        var parent = get_parent()  # Only works in tree
        var children = get_children()  # Only works in tree
        
        # Add children:
        var child = Node.new()
        add_child(child)  # Makes it part of tree hierarchy
    ```

    __Why this matters:__ _If your manager needs to own other nodes or be positioned in the tree hierarchy._

3. You need tree-specific features:

    ```php
    extends Node

    func _ready():
        # Pause mode:
        process_mode = PROCESS_MODE_PAUSABLE
        
        # Tree order matters:
        move_child(some_child, 0)
        
        # Signals that require tree position:
        tree_entered.emit()
        tree_exited.emit()
    ```

4. You're an autoload that needs to persist/manage scene changes:

    ```php
    # This is the REAL reason some autoloads extend Node
    extends Node

    func _ready():
        # Prevent deletion during scene changes:
        process_mode = PROCESS_MODE_ALWAYS
        
        # Manage scene transitions:
        get_tree().current_scene.queue_free()
        # etc.
    ```

## Managers that legitimately need Node:

```php
# SceneManager - manages scene tree directly
extends Node

func change_scene(path: String):
    get_tree().current_scene.queue_free()  # Needs tree access
    var new_scene = load(path).instantiate()
    get_tree().root.add_child(new_scene)  # Needs to add children
```

```php
# TimerManager - needs _process for frame counting
extends Node

var game_time: float = 0.0

func _process(delta):
    game_time += delta
```

```php
# DebugOverlay - needs to be visible in tree
extends CanvasLayer

func _process(delta):
    $FPSLabel.text = "FPS: %d" % Engine.get_frames_per_second()
```

`RefCounted` = "I'm a data/logic manager that exists in memory"

`Node` = "I'm part of the game world/tree and need lifecycle hooks or hierarchy"
