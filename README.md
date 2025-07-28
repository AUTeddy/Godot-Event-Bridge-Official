[![Discord](https://img.shields.io/discord/1399270391226175518?style=for-the-badge&logo=discord)](https://discord.com/channels/1399270391226175518)


If you enjoy using Event-Bridge, please consider supporting my efforts with a donation on Ko-fi 😊 Your contribution will help me continue developing and improving this plugin.

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/M4M51IR6VN)

---

# EventBridge for Godot 4

[![Static Badge](https://img.shields.io/badge/Buy_it_here-red?style=for-the-badge&logo=itchdotio&label=itch.io&labelColor=black)](https://auteddy.itch.io)

**EventBridge** is a lightweight event-driven communication system for Godot 4 that supports:
- **Local and networked events**
- **Signal-based architecture**
- **Automatic JSON event registry**
- **Persistent namespace caching**
- **Duplicate-connection prevention**
- **Easy cleanup with `off_all()`**

Perfect for **multiplayer games** where you need structured, reusable events.

---

## ✅ Features
- **Namespaces** to group related events.
- Event actions:
  - `on(event_name, callable)` → Subscribe
  - `emit(event_name, args)` → Emit locally
  - `remote(event_name, args)` → Emit to server
  - `to_all(event_name, args)` → Emit to all peers
  - `only_me(event_name, args)` → Emit only on local client
  - `to_id(peer_id, event_name, args)` → Emit to a specific peer
- **Offload event cleanup** with `off_all()`
- Editor plugin for managing event registry (`event_registry.json`)

---

## ✅ Installation
1. Copy the **`addons/event_bridge`** folder into your Godot project.
2. Enable the **EventBridge** plugin in **Project Settings → Plugins**.
3. Add **`event_bus.gd`** as an Autoload Singleton:
   - Name: `EventBridge`
   - Path: `res://addons/event_bridge/event_bus.gd`

---

## ✅ Example Event Registry (`event_registry.json`)
```json
{
	"Test": [
		{
			"name": "ping",
			"rpc": "remote",
			"args": ["msg"]
		},
		{
			"name": "pong",
			"rpc": "remote",
			"args": ["msg"]
		}
	]
}
```

---

## ✅ Usage

### 1. Get a Namespace
```gdscript
var ns = EventBridge.get_namespace("Test")
```

### 2. Subscribe to Events
```gdscript
ns.on("ping", func(msg):
	print("[CLIENT] PING received: %s" % msg)
	ns.remote("pong", ["Hello back from CLIENT %d" % multiplayer.get_unique_id()])
)
```

### 3. Emit Events (Server Example)
```gdscript
if multiplayer.is_server():
	await get_tree().create_timer(2.0).timeout
	ns.to_all("ping", ["Hello from SERVER"])
```

---

## ✅ Cleanup
When a node is removed, disconnect all its event handlers:
```gdscript
func _exit_tree():
	ns.off_all()
```

---

## ✅ Full API
| Method                | Description                                     |
|----------------------|-------------------------------------------------|
| `get_namespace(name)`| Returns Namespace object for a category         |
| `on(event, callable)`| Subscribe to an event                          |
| `emit(event, args)`  | Emit locally                                   |
| `remote(event, args)`| Emit to server                                 |
| `to_all(event, args)`| Emit to all peers                              |
| `only_me(event, args)`| Emit only on this client                      |
| `to_id(id, event, args)`| Emit to specific peer                       |
| `off_all()`          | Remove all handlers in this namespace          |

---

## ✅ Internal Structure
- **Signals are dynamically registered** based on the event registry.
- **Namespace caching** avoids creating duplicates.
- **Connected handler tracking** ensures no duplicate signal connections.

---

## ✅ Example Output
```
[EventBridge] Loaded registry from: res://addons/event_bridge/event_registry.json
[EventBridge] EventBus initialized with namespaces: ["Test"]
[EventBridge] Connected handler for Test::ping
[EventBridge] Connected handler for Test::pong
[EventBridge] Emit to all: Test::ping ["Hello from SERVER"]
[EventBridge] RPC received: Test::ping ["Hello from SERVER"]
[CLIENT] PING received: Hello from SERVER
[SERVER] PONG received from client: Hello back from CLIENT 123456789
```

---

## ✅ License

EventBridge Plugin Commercial License
Version 1.0 – 2025

Copyright (c) 2025 [Your Name or Company]

Permission is granted to the purchaser to use the EventBridge plugin under the following terms:

1. Grant of License
You are granted a non-exclusive, non-transferable license to use this plugin in an unlimited number of personal and commercial projects.

2. Allowed Uses
- You may use the plugin in games or applications you develop for personal or commercial purposes.
- You may modify the plugin for use within your own projects.

3. Restrictions
- Redistribution, resale, or sharing of the plugin source code (original or modified) is strictly prohibited.
- You may not include the plugin in any public repository or open-source project without prior written permission.
- You may not sublicense or repackage the plugin as part of another asset or product for distribution.

4. Ownership
This license does not grant ownership of the plugin. All rights, title, and interest remain with the copyright holder.

5. Liability & Warranty
The plugin is provided "as-is" without any warranties. The author is not liable for any damages arising from the use of the plugin.

By purchasing or downloading this plugin, you agree to these terms.

---

## ✅ Roadmap
- [ ] Add GUI editor for event registry
- [ ] Add argument validation on emit
- [ ] Add type hints for callable arguments
