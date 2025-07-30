EventBridge – Multiplayer Event System
EventBridge is a Godot plugin that provides a high-level event-based API for multiplayer games, powered by Godot’s RPC system and a visual editor.

How It Works
Define your multiplayer events visually in the EventBridge Dock.

Generate the Event Registry and EventManager.gd API file.

Use the generated API in your code instead of writing raw rpc() calls.

Schema Overview
Each event has the following fields:

Field	Description
name	Event name (used in code).
target	How the event is delivered:
emit_local – local only
to_server – send to server
to_all – broadcast to all
to_id – send to specific peer
mode	Who is allowed to send this RPC:
authority (usually server)
any_peer
sync	Whether to call locally:
call_local or call_remote.
transfer_mode	Network delivery method:
reliable (safe)
unreliable (fast, can drop)
unreliable_ordered (fast but ordered)
transfer_channel	Integer channel (separate streams to avoid packet ordering conflicts).
args	List of typed arguments (String, int, float, etc.).

Example Schema
json
Copy
Edit
{
  "Game": [
    {
      "name": "dice_roll",
      "target": "to_all",
      "mode": "authority",
      "sync": "call_local",
      "transfer_mode": "unreliable",
      "transfer_channel": 1,
      "args": [
        { "name": "player_id", "type": "int" },
        { "name": "value", "type": "int" }
      ]
    }
  ]
}
Workflow
Open EventBridge Dock in the Godot editor.

Create categories and events.

Configure networking settings per event.

Click “Generate” → This updates:

event_registry.json

EventManager.gd (API for your game).

Restart your game before peers connect.

Important
Godot computes an RPC checksum at connection time.
Therefore:

You cannot add new events or change RPC settings after clients are connected.

Always regenerate and restart before connecting peers.

Example Usage
gdscript
Copy
Edit
# Subscribing
EventManager.on_dice_roll(func(player_id: int, value: int):
    print("Dice rolled by", player_id, "=", value)
)

# Emitting
EventManager.dice_roll(player_id, 6)
