# EventBridge for Godot 4
[![Discord](https://img.shields.io/discord/1399270391226175518?style=for-the-badge&logo=discord)](https://discord.com/channels/1399270391226175518)
[![Static Badge](https://img.shields.io/badge/Buy_on-itch.io-red?style=for-the-badge&logo=itchdotio&labelColor=black)](https://auteddy.itch.io)
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/M4M51IR6VN)

---

## 🚀 The Smartest Event System for Godot 4

Tired of **signal spaghetti** and **RPC chaos**?  
**EventBridge** gives you a **clean, scalable, and multiplayer-ready** event-driven architecture for Godot 4.  
Perfect for **single-player and multiplayer** projects.

---

## ✅ Why Choose EventBridge?
- **Local & Network Events** – One API for local, server, peer-specific, or broadcast events  
- **Namespaces for Organization** – Keep events modular and maintainable  
- **Automatic Event Registry** – Define once, reuse everywhere  
- **Editor Integration** – Manage events visually with the built-in Godot panel  
- **Duplicate Protection** – No more double signal connections  
- **Persistent Caching** – Fast lookups even for large projects  
- **One-Line Cleanup** – `off_all()` removes dangling handlers instantly  

---

## 🎮 Perfect for Multiplayer
Send events:
✔ **To the server**  
✔ **To all peers**  
✔ **To a specific peer**  

**Example:**
```gdscript
var ns = EventBridge.get_namespace("Lobby")

ns.on("ping", func(msg):
    print("PING:", msg)
)

ns.remote("pong", ["Hello from client!"])
```

---

## 📦 Installation
1. Copy the **`addons/event_bridge`** folder into your Godot project  
2. Enable the plugin in **Project Settings → Plugins**  
3. Add **`event_bus.gd`** as an **Autoload Singleton**:
   - Name: `EventBridge`
   - Path: `res://addons/event_bridge/event_bus.gd`  

---

## ⚡ Quick Example Event Registry
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

## 🛠 Usage
### 1. Get a Namespace
```gdscript
var ns = EventBridge.get_namespace("Test")
```

### 2. Subscribe to Events
```gdscript
ns.on("ping", func(msg):
    print("[CLIENT] PING:", msg)
    ns.remote("pong", ["Hello from CLIENT %d" % multiplayer.get_unique_id()])
)
```

### 3. Emit Events (Server)
```gdscript
if multiplayer.is_server():
    await get_tree().create_timer(2.0).timeout
    ns.to_all("ping", ["Hello from SERVER"])
```

### ✅ Cleanup
```gdscript
func _exit_tree():
    ns.off_all()
```

---

## 🔑 Full API
| Method                  | Description                                    |
|-------------------------|-----------------------------------------------|
| `get_namespace(name)`   | Returns Namespace object for a category      |
| `on(event, callable)`   | Subscribe to an event                        |
| `emit(event, args)`     | Emit locally                                  |
| `remote(event, args)`   | Emit to server                                |
| `to_all(event, args)`   | Emit to all peers                             |
| `only_me(event, args)`  | Emit only on this client                      |
| `to_id(id, event, args)`| Emit to a specific peer                       |
| `off_all()`             | Remove all handlers in this namespace         |

---

## 📢 Example Output
```
[EventBridge] Loaded registry from: res://addons/event_bridge/event_registry.json
[EventBridge] EventBus initialized with namespaces: ["Test"]
[EventBridge] Connected handler for Test::ping
[EventBridge] Emit to all: Test::ping ["Hello from SERVER"]
[CLIENT] PING received: Hello from SERVER
[SERVER] PONG received from client: Hello back from CLIENT 123456789
```

---

## 📄 License
**EventBridge Plugin Commercial License** (included in package):  
✔ Use in unlimited projects (personal & commercial)  
❌ No redistribution or resale  

---

## 🛣 Roadmap
- [ ] GUI editor for event registry  
- [ ] Argument validation on emit  
- [ ] Type hints for callable arguments  

---

## 💬 Join the Community
[![Discord](https://img.shields.io/discord/1399270391226175518?style=for-the-badge&logo=discord)](https://discord.com/channels/1399270391226175518)

---

### ❤️ Support Development
If you enjoy using EventBridge, consider supporting my work:  
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/M4M51IR6VN)

---

[![Static Badge](https://img.shields.io/badge/Buy_on-itch.io-red?style=for-the-badge&logo=itchdotio&labelColor=black)](https://auteddy.itch.io)
