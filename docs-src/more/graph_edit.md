# Working with GraphEdit Control

`GraphEdit` is a `Control` for building node-graph editors. Its children are usually `GraphNode` nodes. Connections are managed by the `GraphEdit` with the `connection_request` signal.

## GraphEdit Control

- `GraphEdit` — Holds the graph, handles pan/zoom/connections.
- `GraphNode` — A draggable, resizable node with input/output slots.

## Node Types

Both are built-in `Control` nodes:

- `GraphEdit` is the canvas/viewer.
- `GraphNode` is the thing users drag around.
- `class_name MyNode extends GraphNode` lets you make custom node types.

## Node Parameters

- `title` — Header text.
- `resizable` — Can the user resize the node.
- `show_close` — Show a close button.
- `selected` — Whether the node is selected.
- `position_offset` — Graph-space position.

## Node Behaviors

`GraphNode` exposes slots based on its children. Use `set_slot(index, enable_left, type_left, color_left, enable_right, type_right, color_right)` to mark a child as a left input, right output, or both. Nodes can be dragged, selected, and resized.

## Node Effects

Style nodes with `theme_override_*` properties or by adding child `Control` nodes. Slots can be color-coded by connection type.

## Connection Types

Each slot has a `type` (an `int`). Connections are only meaningful to your code; Godot lets you connect any types. Validate in `connection_request` to enforce type rules.

## Connection Parameters

`GraphEdit.connect_node(from_node, from_port, to_node, to_port)` creates a connection. `get_connection_list()` returns an `Array` of dictionaries with `from`, `from_port`, `to`, and `to_port` keys.

## Connection Behaviors

- `connection_request(from_node, from_port, to_node, to_port)` — Fired when the user drags a connection.
- `disconnection_request(...)` — Fired when the user removes one.
- `delete_nodes_request(nodes)` — Fired when the user tries to delete selected nodes.

## Connection Effects

Draw custom connection lines by overriding `GraphEdit._draw` or use `theme_override_*` for line/wire color. Node `slot` colors also tint the connection endpoints.

## Example: Simple graph

```gd
# ============================================
# graph_edit.gd — simple two-node graph
# ============================================
extends GraphEdit

func _ready():
    connection_request.connect(_on_connection_request)

    # Node A has an output
    var node_a = GraphNode.new()
    node_a.title = "A"
    var label_a = Label.new()
    label_a.text = "Out"
    node_a.add_child(label_a)
    node_a.set_slot(0, false, 0, Color.WHITE, true, 0, Color.RED)
    add_child(node_a)

    # Node B has an input
    var node_b = GraphNode.new()
    node_b.title = "B"
    var label_b = Label.new()
    label_b.text = "In"
    node_b.add_child(label_b)
    node_b.set_slot(0, true, 0, Color.BLUE, false, 0, Color.WHITE)
    add_child(node_b)

    # Arrange them visually
    node_a.position_offset = Vector2(40, 40)
    node_b.position_offset = Vector2(280, 40)

func _on_connection_request(from_node, from_port, to_node, to_port):
    if not is_node_connected(from_node, from_port, to_node, to_port):
        connect_node(from_node, from_port, to_node, to_port)
```

## Example: Complex graph

```gd
# ============================================
# graph_edit.gd — save/load the graph
# ============================================
extends GraphEdit

func save_graph() -> Array:
    var data = []
    for connection in get_connection_list():
        data.append({
            "from": connection.from,
            "from_port": connection.from_port,
            "to": connection.to,
            "to_port": connection.to_port
        })
    return data

func load_graph(connections: Array):
    # Clear existing connections
    for connection in get_connection_list():
        disconnect_node(connection.from, connection.from_port, connection.to, connection.to_port)

    for c in connections:
        connect_node(c["from"], c["from_port"], c["to"], c["to_port"])
```

## Example: Graph with custom nodes

```gd
# ============================================
# number_node.gd
# ============================================
class_name NumberNode extends GraphNode

func _init():
    title = "Number"
    resizable = false

    var line = LineEdit.new()
    line.text = "0"
    add_child(line)

    # Output slot on the right of the LineEdit
    set_slot(0, false, 0, Color.WHITE, true, 1, Color.GREEN)
```

__Usage:__

```gd
var n = NumberNode.new()
n.position_offset = Vector2(100, 100)
add_child(n)
```

## Example: Graph with custom connections

```gd
# ============================================
# graph_edit.gd — connect by node type
# ============================================
extends GraphEdit

func _ready():
    connection_request.connect(_on_connection_request)

func _on_connection_request(from_node, from_port, to_node, to_port):
    var src = get_node(NodePath(from_node))
    var dst = get_node(NodePath(to_node))

    # Only NumberNode -> MathNode is allowed
    if src is NumberNode and dst is MathNode:
        if not is_node_connected(from_node, from_port, to_node, to_port):
            connect_node(from_node, from_port, to_node, to_port)
```
