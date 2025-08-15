# EventBridge Advanced Tips & Best Practices

This guide expands on **performance optimizations**, **networking best practices**, **security measures**, and **advanced event configuration** for developers using EventBridge in Godot.

---

## 🚀 Performance Optimization

### 1. **Batching Events**
- Instead of sending many tiny events in quick succession, combine them into a single event using an `Array` or `Dictionary` payload.
- Greatly reduces **RPC overhead** and improves performance when syncing frequent updates such as player positions or scores.

**Example:**
```gdscript
# Combine multiple position updates into one packet
var updates = [
    {"id": 1, "pos": Vector2(100, 200)},
    {"id": 2, "pos": Vector2(300, 450)}
]
Game.to_all("batched_positions", [updates])
```

---

### 2. **Avoid High-Frequency Reliable RPC**
- Use `reliable` for **critical events only** (turn changes, combat results).
- For frequent updates (movement, animations), use:
  - `unreliable` or `unreliable_ordered` (when slight packet loss is acceptable).
- Reliable mode can cause delays if a packet is lost, as all later packets wait for retransmission.

**Example:**
```gdscript
Test.to_all_unreliable("update_position", [x, y])
```

---

## 🌐 Networking Best Practices

### 3. **Transfer Channels**
Organize events by priority using **channels**:

| Channel | Use Case |
|---------|----------|
| **0**   | Highest priority (critical game events) |
| **1–2** | Secondary traffic (UI sync, chat) |
| **3–4** | Low-priority updates (animations, effects) |

Using separate channels prevents important packets from being delayed by large, low-priority messages.

**Example:**
```gdscript
Test.to_all("ui_sync", [data], channel=1)
```

---

### 4. **Use ENet’s Built-in Features**
Godot’s `NetworkedMultiplayerENet` supports:

- **Compression** → Reduces bandwidth usage for large payloads.
- **Packet Merging** → Combine small packets into fewer transmissions (`enet_compress` in Project Settings).
- **Custom Channel Count** → If your game needs more than 4 channels:
```gdscript
peer.set_channel_count(5)
```

---

## 🔐 Security & Authority Control

### 5. **Authority Mode**
- In a **server-authoritative** design, the server makes all final decisions.
- Clients send `to_server` events for requests; server validates and executes.
- Prevents cheating by never letting clients decide outcomes.

**Example:**
```gdscript
# Client side
Game.to_server("request_attack", [target_id])

# Server side
func _on_request_attack(peer_id, target_id):
    if validate_attack(peer_id, target_id):
        Game.to_all("attack_success", [peer_id, target_id])
```

---

### 6. **Prevent RPC Injection**
- Never trust events from `any_peer` for sensitive actions.
- Use authority checks to ensure only the server can emit critical events.

---

## 🎛 Debugging & Logging

### 7. **Enable Debug Mode**
- Toggle **Debug** in the EventBridge dock.
- Enables toast notifications when events are auto-corrected.
- Prints event mappings and RPC usage in the Output console.

---

### 8. **Trace Event Usage**
- Use **Connections View** in the plugin to see all active event subscriptions and emissions.
- Double-click entries to jump to the exact script line.

---

## ✅ Quick Checklist

✔ Use **emit_local** for offline/local logic.  
✔ Use **to_server** for client → server communication.  
✔ Always **validate critical actions** on the server.  
✔ Batch frequent updates & prefer `unreliable` for non-critical traffic.  
✔ Separate **channels** by priority to prevent delays.  
✔ Enable **debug mode** during development for better visibility.  

---

**Pro Tip:** Combine these strategies for maximum performance and security without sacrificing development speed.
