1) Button pressed → send an event
Manual (namespace API)
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

Auto‑generated (EventManager)
# UI.gd
@onready var play_btn: Button = %PlayButton

func _ready() -> void:
    play_btn.pressed.connect(_on_play_pressed)
    EventManager.on_play_clicked(func():
        print("[ALL] Someone clicked Play")
    )

func _on_play_pressed() -> void:
    if multiplayer.is_server():
        EventManager.play_clicked()      # broadcast, per config
    else:
        EventManager.play_clicked()      # goes to server if configured that way


(In your registry, set UI.play_clicked target appropriately: to_server if you want a server-authoritative flow, or to_all if server will rebroadcast.)

2) Area2D entered → client asks server for a boost → server applies boost to that player
Event design (example)

Game.request_boost(player_id) — to_server

Player.apply_boost(player_scale, peer_id) — to_id (server → specific client)

Manual (namespace API)

Client side (the Area2D)

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


Server side

# ServerBoost.gd (autoload or server node)
var ns_game := EventBridge.get_namespace("Game")
var ns_player := EventBridge.get_namespace("Player")

func _ready() -> void:
    # Server handles boost requests
    ns_game.on("request_boost", func(player_id):
        # do validation, cooldowns, etc. then apply boost
        var peer_id := _peer_id_from_player_id(player_id) # your mapping
        var new_scale := Vector2(1.5, 1.5)
        ns_player.to_id(peer_id, "apply_boost", [new_scale])
    )

func _peer_id_from_player_id(player_id: int) -> int:
    # implement your lookup
    return player_id


Client side (player applies the boost)

# Player.gd
var ns_player := EventBridge.get_namespace("Player")

func _ready() -> void:
    ns_player.on("apply_boost", func(new_scale: Vector2):
        $Sprite2D.scale = new_scale
        # or any boost effect you want
    )

Auto‑generated (EventManager)

Client (Area2D)

# PowerZone.gd
func _ready() -> void:
    body_entered.connect(_on_body_entered)

func _on_body_entered(body: Node) -> void:
    if not body or not body.has_method("get_player_id"):
        return
    EventManager.request_boost(body.get_player_id())   # configured as to_server


Server

# ServerBoost.gd
func _ready() -> void:
    EventManager.on_request_boost(func(player_id: int):
        var peer_id := _peer_id_from_player_id(player_id)
        EventManager.apply_boost(Vector2(1.5, 1.5), peer_id)  # configured as to_id
    )

func _peer_id_from_player_id(player_id: int) -> int:
    return player_id


Client (player)

# Player.gd
func _ready() -> void:
    EventManager.on_apply_boost(func(scale: Vector2):
        $Sprite2D.scale = scale
    )

3) Button toggles ready state → server broadcasts lobby update
Event design

Lobby.set_ready(player_id, is_ready) — to_server

Lobby.ready_state_changed(player_id, is_ready) — to_all

Client

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


Server

# LobbyServer.gd
var ns := EventBridge.get_namespace("Lobby")

func _ready() -> void:
    ns.on("set_ready", func(pid: int, state: bool):
        # update server data store then inform everyone
        ns.to_all("ready_state_changed", [pid, state])
    )

4) Area2D exit → client asks server to remove effect → server confirms to that player
Event design

Game.request_remove_boost(player_id) — to_server

Player.clear_boost(peer_id) — to_id

# PowerZone.gd (Area2D)
func _ready() -> void:
    body_exited.connect(_on_body_exited)

func _on_body_exited(body: Node) -> void:
    if body and body.has_method("get_player_id"):
        EventManager.request_remove_boost(body.get_player_id())  # to_server

# ServerBoost.gd
func _ready() -> void:
    EventManager.on_request_remove_boost(func(player_id: int):
        EventManager.clear_boost(_peer_id_from_player_id(player_id))  # to_id
    )

# Player.gd
func _ready() -> void:
    EventManager.on_clear_boost(func():
        $Sprite2D.scale = Vector2.ONE
    )

Tips

Use server‑authoritative flows for anything that affects game state (like boosts).

Keep your registry clear: name events by intent (request_boost, apply_boost) and set targets:

Client → Server: to_server

Server → One client: to_id

Server → Everyone: to_all

Local only: emit_local

Clean up listeners (off_* or ns.off_all()) when nodes are freed.
