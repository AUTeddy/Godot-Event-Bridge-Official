
# EventBridge Usage Recipes — UI, Areas, and Lobby Flows (Godot 4)

This guide gives **copy‑paste ready** examples for common gameplay interactions using **EventBridge**.  
Each recipe shows both the **manual Namespace API** and the **auto‑generated EventManager API** variants, plus **EventDock** configuration and **validator suggestions**.

> Assumptions: You’ve installed the plugin, generated your registry, and your **server is running**.

---

## 🟢 1) Button pressed → send an event

### EventDock configuration (UI.play_clicked)
| Property | Value | Notes |
|---|---|---|
| Namespace | `UI` |  |
| Event | `play_clicked` |  |
| Target | `to_server` *(recommended)* | Server‑authoritative: client requests → server rebroadcasts |
| Mode | `authority` | Required for `to_server` |
| Sync | `call_remote` | Networked |
| Transfer | `reliable` | Important UI intent |
| Channel | `0` | Critical/UI layer |

> If you want the server to broadcast directly, configure `Target=to_all` and emit only on the server.

### Manual (Namespace API)
```gdscript
# UI.gd (attached to a scene with a Button named "PlayButton")
@onready var play_btn: Button = %PlayButton
var ns := EventBridge.get_namespace("UI")

func _ready() -> void:
    play_btn.pressed.connect(_on_play_pressed)
    ns.on("play_clicked", func():
        print("[ALL] Someone clicked Play")
    )

func _on_play_pressed() -> void:
    if multiplayer.is_server():
        # Server announces to all peers
        ns.to_all("play_clicked")
    else:
        # Client asks server to announce (or handle) it
        ns.to_server("play_clicked")
```

### Auto‑generated (EventManager API)
```gdscript
# UI.gd
@onready var play_btn: Button = %PlayButton

func _ready() -> void:
    play_btn.pressed.connect(_on_play_pressed)
    EventManager.on_play_clicked(func():
        print("[ALL] Someone clicked Play")
    )

func _on_play_pressed() -> void:
    # The actual path depends on your registry config for UI.play_clicked
    EventManager.play_clicked()
```

**Validator suggestion (server):**
```gdscript
func validate_play_clicked(args: Array) -> bool:
    # Optionally gate by UI state or eligibility
    return true
```

---

## ⚡ 2) Area2D entered → client asks server for a boost → server applies boost to that player

### Event design
- `Game.request_boost(player_id)` — **to_server** (client → server)
- `Player.apply_boost(player_scale, peer_id)` — **to_id** (server → specific client)

### EventDock configurations
**Game.request_boost**
| Property | Value | Notes |
|---|---|---|
| Namespace | `Game` |  |
| Event | `request_boost` |  |
| Target | `to_server` | Client request |
| Mode | `authority` | Required |
| Sync | `call_remote` |  |
| Transfer | `reliable` | Important state change |
| Channel | `0` | Critical |

**Player.apply_boost**
| Property | Value | Notes |
|---|---|---|
| Namespace | `Player` |  |
| Event | `apply_boost` |  |
| Target | `to_id` | Server → one client |
| Mode | `any_peer` | Emitted by the server |
| Sync | `call_remote` |  |
| Transfer | `reliable` | State change |
| Channel | `0` | Critical |

### Manual (Namespace API)

**Client (Area2D)**
```gdscript
# PowerZone.gd (Area2D)
var ns_game := EventBridge.get_namespace("Game")

func _ready() -> void:
    body_entered.connect(_on_body_entered)

func _on_body_entered(body: Node) -> void:
    if not body or not body.has_method("get_player_id"):
        return
    var pid = body.get_player_id()  # your player script should expose this
    # Ask server for a boost
    ns_game.to_server("request_boost", [pid])
```

**Server handler**
```gdscript
# ServerBoost.gd (autoload or server node)
var ns_game := EventBridge.get_namespace("Game")
var ns_player := EventBridge.get_namespace("Player")

func _ready() -> void:
    # Server handles boost requests
    ns_game.on("request_boost", func(player_id):
        # TODO: validation, cooldowns, etc.
        var peer_id := _peer_id_from_player_id(player_id) # your mapping
        var new_scale := Vector2(1.5, 1.5)
        ns_player.to_id(peer_id, "apply_boost", [new_scale])
    )

func _peer_id_from_player_id(player_id: int) -> int:
    # implement your lookup
    return player_id
```

**Client (Player applies boost)**
```gdscript
# Player.gd
var ns_player := EventBridge.get_namespace("Player")

func _ready() -> void:
    ns_player.on("apply_boost", func(new_scale: Vector2):
        $Sprite2D.scale = new_scale
        # or any boost effect you want
    )
```

### Auto‑generated (EventManager API)

**Client (Area2D)**
```gdscript
# PowerZone.gd
func _ready() -> void:
    body_entered.connect(_on_body_entered)

func _on_body_entered(body: Node) -> void:
    if not body or not body.has_method("get_player_id"):
        return
    EventManager.request_boost(body.get_player_id())   # to_server
```

**Server**
```gdscript
# ServerBoost.gd
func _ready() -> void:
    EventManager.on_request_boost(func(player_id: int):
        var peer_id := _peer_id_from_player_id(player_id)
        EventManager.apply_boost(Vector2(1.5, 1.5), peer_id)  # to_id
    )

func _peer_id_from_player_id(player_id: int) -> int:
    return player_id
```

**Client (player)**
```gdscript
# Player.gd
func _ready() -> void:
    EventManager.on_apply_boost(func(scale: Vector2):
        $Sprite2D.scale = scale
    )
```

**Validator suggestions (server):**
```gdscript
func validate_request_boost(args: Array) -> bool:
    return args.size() == 1 and typeof(args[0]) == TYPE_INT

func validate_apply_boost(args: Array) -> bool:
    return args.size() == 2 and args[0] is Vector2 and typeof(args[1]) == TYPE_INT
```

---

## 🧩 3) Button toggles ready state → server broadcasts lobby update

### Event design
- `Lobby.set_ready(player_id, is_ready)` — **to_server**
- `Lobby.ready_state_changed(player_id, is_ready)` — **to_all**

### EventDock configurations
**Lobby.set_ready**
| Property | Value | Notes |
|---|---|---|
| Namespace | `Lobby` |  |
| Event | `set_ready` |  |
| Target | `to_server` | Client request |
| Mode | `authority` | Required |
| Sync | `call_remote` |  |
| Transfer | `reliable` | State |
| Channel | `0` | Critical |

**Lobby.ready_state_changed**
| Property | Value | Notes |
|---|---|---|
| Namespace | `Lobby` |  |
| Event | `ready_state_changed` |  |
| Target | `to_all` | Server broadcast |
| Mode | `any_peer` | Emitted by the server |
| Sync | `call_remote` |  |
| Transfer | `reliable` | State |
| Channel | `0` | Critical |

### Client
```gdscript
# ReadyButton.gd
@onready var btn: Button = %ReadyButton
var ns := EventBridge.get_namespace("Lobby")
var _is_ready := false

func _ready() -> void:
    btn.pressed.connect(func():
        _is_ready = !_is_ready
        ns.to_server("set_ready", [multiplayer.get_unique_id(), _is_ready])
    )

    ns.on("ready_state_changed", func(pid: int, state: bool):
        print("Player", pid, "ready =", state)
    )
```

### Server
```gdscript
# LobbyServer.gd
var ns := EventBridge.get_namespace("Lobby")

func _ready() -> void:
    ns.on("set_ready", func(pid: int, state: bool):
        # update server data store then inform everyone
        ns.to_all("ready_state_changed", [pid, state])
    )
```

**Validator suggestions (server):**
```gdscript
func validate_set_ready(args: Array) -> bool:
    return args.size() == 2 and typeof(args[0]) == TYPE_INT and typeof(args[1]) == TYPE_BOOL
```

---

## 🧼 4) Area2D exit → client asks server to remove effect → server confirms to that player

### Event design
- `Game.request_remove_boost(player_id)` — **to_server**
- `Player.clear_boost(peer_id)` — **to_id**

### EventDock configurations
**Game.request_remove_boost**
| Property | Value | Notes |
|---|---|---|
| Namespace | `Game` |  |
| Event | `request_remove_boost` |  |
| Target | `to_server` | Client request |
| Mode | `authority` | Required |
| Sync | `call_remote` |  |
| Transfer | `reliable` | State |
| Channel | `0` | Critical |

**Player.clear_boost**
| Property | Value | Notes |
|---|---|---|
| Namespace | `Player` |  |
| Event | `clear_boost` |  |
| Target | `to_id` | Server → client |
| Mode | `any_peer` | Emitted by the server |
| Sync | `call_remote` |  |
| Transfer | `reliable` | State |
| Channel | `0` | Critical |

### Client (Area2D)
```gdscript
# PowerZone.gd (Area2D)
func _ready() -> void:
    body_exited.connect(_on_body_exited)

func _on_body_exited(body: Node) -> void:
    if body and body.has_method("get_player_id"):
        EventManager.request_remove_boost(body.get_player_id())  # to_server
```

### Server
```gdscript
# ServerBoost.gd
func _ready() -> void:
    EventManager.on_request_remove_boost(func(player_id: int):
        EventManager.clear_boost(_peer_id_from_player_id(player_id))  # to_id
    )
```

### Client (Player)
```gdscript
# Player.gd
func _ready() -> void:
    EventManager.on_clear_boost(func():
        $Sprite2D.scale = Vector2.ONE
    )
```

**Validator suggestions (server):**
```gdscript
func validate_request_remove_boost(args: Array) -> bool:
    return args.size() == 1 and typeof(args[0]) == TYPE_INT

func validate_clear_boost(args: Array) -> bool:
    return args.size() == 1 and typeof(args[0]) == TYPE_INT
```

---

## 🧹 Cleanup & Lifecycle

- For Namespace API: `ns.off("event", callable)` or `ns.off_all()` in `_exit_tree()`.
- For EventManager API: use the matching `off_<event>(callable)` if generated, or keep references to callbacks.

```gdscript
func _exit_tree() -> void:
    EventManager.off_ready_state_changed(_on_ready_changed) # if available
    # or keep a namespace handle and call ns.off_all()
```

---

## 🧪 Testing Checklist

- Server running? **Yes**
- Registry generated and synced on all peers? **Yes**
- Validators added for `to_server` events? **Yes**
- Correct **transfer modes** and **channels**? **Yes**
- Debug enabled in EventDock for toasts/logs while iterating? **Yes**

---

## 📝 Notes & Pitfalls

- Prefer **server‑authoritative** designs. Clients **request**, server **decides** and **broadcasts**.
- Don’t mix unrelated events on the same **unreliable_ordered** channel.
- Keep validators **fast**. Heavy checks belong in game logic after acceptance.
- When refactoring, **Regenerate** the registry/API and restart server/clients to avoid mismatches.

---

Happy shipping! 🚀
