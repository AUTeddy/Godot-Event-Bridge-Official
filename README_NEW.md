# EventBridge for Godot 4

EventBridge is a Godot 4 editor plugin that provides a unified event-driven system for both single-player and multiplayer games. It simplifies signal handling and RPC calls by letting developers define events once, generate a ready-to-use API, and control all networking settings visually. EventBridge uses [Godot MultiplayerAPI](https://docs.godotengine.org/en/stable/classes/class_multiplayerapi.html) internally, ensuring full compliance with networking best practices.

---

## ✅ Why Use EventBridge?

### Common Issues with Native Godot Networking
Using [Godot MultiplayerAPI](https://docs.godotengine.org/en/stable/classes/class_multiplayerapi.html) or [MultiplayerAPIExtension](https://docs.godotengine.org/en/stable/classes/class_multiplayerapiextension.html) directly:
- Requires annotating each method with `@rpc` and setting configuration manually.
- Calls like `rpc()`, `rpc_id()`, and `rpc_config()` are spread across code.
- Developers must manually enforce authority rules and optimize transfer settings.

Example:
```gdscript
@rpc("any_peer", TransferMode.RELIABLE, 0)
func ping(msg: String):
    print("PING:", msg)

rpc_id(1, "ping", "Hello from client!")
This approach becomes difficult to maintain as projects scale.

EventBridge Solution
EventBridge automates these steps:

Define events visually in a dock panel.

Configure networking options with dropdowns.

Click Generate → Instantly get an API in EventManager.gd.

Example:

gdscript
Kopieren
Bearbeiten
# Auto-generated API
EventManager.ping("Hello from client!")
No manual rpc() calls. EventBridge:

Dynamically calls rpc_config() based on your schema.

Handles authority, reliability, and channels automatically.

Applies safe defaults and auto-corrects invalid configurations.

Clear Benefits
✔ Centralized Networking Rules in event_registry.json
✔ Auto-Generated GDScript API for clean usage
✔ Works with Godot MultiplayerAPI out of the box
✔ Auto-correction for invalid setups
✔ Built-in debug tools and event map

✅ Developer Recap
Previously:
Events used a single rpc flag (remote, to_all, to_id, local).

Networking logic was inconsistent.

Developers needed deep RPC knowledge to optimize performance.

Now:
Structured event schema for full control.

Human-readable config: target, mode, sync, transfer_mode, transfer_channel.

Auto-generated API: EventManager.gd.

Dynamic rpc_config() → No manual setup.

✅ What Changed
Old schema:

json
Kopieren
Bearbeiten
{ "name": "ping", "rpc": "remote", "args": [] }
New schema:

json
Kopieren
Bearbeiten
{
  "name": "ping",
  "target": "to_server",
  "mode": "any_peer",
  "sync": "call_local",
  "transfer_mode": "reliable",
  "transfer_channel": 0,
  "args": [{ "name": "msg", "type": "String" }]
}
Why This Matters
Target → Who receives the event (emit_local, to_server, to_all, to_id).

Mode → Who can emit it (authority, any_peer).

Sync → Local or remote call.

Transfer Mode → Reliable, unreliable, or ordered for performance tuning.

Transfer Channel → Prioritize traffic streams.

✅ Key Features
Visual Event Editor in Godot.

Persistent schema in JSON (event_registry.json).

Auto-corrected invalid settings.

Namespaces for modular event grouping.

Debug tools with usage maps.

✅ Changelog Highlights (v2.0)
Full RPC configuration support: target, mode, sync, transfer_mode, transfer_channel.

Updated dock UI with dropdowns, spinboxes, and live preview.

Auto-generated EventManager with documentation and hints.

Runtime validation and auto-fixes.

Backward compatibility for old API calls.

✅ Valid Configurations Table
Target	Mode	Sync	Transfer Mode	Channel	Notes
emit_local	any_peer	call_local	reliable	0–4	Local events only.
to_server	authority	call_remote	reliable	0–4	Client → Server.
to_id	any_peer	call_remote	reliable	0–4	Peer → Peer messaging.
to_all	any_peer	call_remote	reliable	0–4	Broadcast to all peers.

✅ Auto-Correction Flow (Summary)
emit_local → Forces call_local, any_peer, reliable.

to_server → Forces authority and call_remote.

to_all/to_id → Forces call_remote.

Channels are clamped between 0–4.

✅ Project Structure
plaintext
Kopieren
Bearbeiten
addons/
└── event_bridge/
    ├── assets/        # Icons for the dock UI
    ├── generated/     # Auto-generated files (EventManager.gd, event_registry.json)
    ├── event_dock.gd  # Main plugin UI logic
    ├── event_bus.gd   # Core runtime for event handling
    ├── event_bridge.gd# Plugin entry point
docs/
images/
✅ External References
Godot MultiplayerAPI

Godot MultiplayerAPIExtension

Godot EditorPlugin API

✅ About Me
I built EventBridge to solve recurring pain points in Godot networking:

Scattered RPC logic across code.

Manual rpc_config() maintenance.

Lack of a central event-driven pattern for multiplayer games.
