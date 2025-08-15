# EventBridge Advanced Tips

This guide covers **performance optimizations**, **networking best practices**, and **advanced event configuration**.

---

## 🚀 Performance Optimization

### 1. **Batching Events**
- Combine multiple small events into a single event with an `Array` or `Dictionary` payload.
- Reduces RPC overhead when sending frequent updates (e.g., positions).

### 2. **Avoid High-Frequency Reliable RPC**
- Use `reliable` for critical events only (turn changes, actions).
- For frequent updates (movement, animation), use:
  - `unreliable` or `unreliable_ordered` *(if network latency allows)*.

- Example:
  ``gdscript
  Test.to_all_unreliable("update_position", [x, y])
  ``

## 🌐 Networking Best Practices

### **3. Transfer Channels**
Use channels to prioritize important traffic:

- **Channel 0** → Highest priority (critical events).
- **Channel 1–4** → Secondary traffic (UI sync, animations).
- Separate channels avoid packet queuing delays.

---

### **4. Use ENet’s Features**
Godot’s `NetworkedMultiplayerENet` supports:

- **Compression** → Reduces bandwidth usage.
- **Packet Merging** → Enable via `enet_compress` in Project Settings.
- Use `peer.set_channel_count(5)` if you need more than 4 channels.

---

## 🔐 Security & Authority Control

### **5. Authority Mode**
Server authoritative design:

- Only the server can trigger critical events (combat results, turn change).
- Clients emit only `to_server` calls.
- Validate all client events server-side before applying game logic.

---

### **6. Prevent RPC Injection**
Never trust `any_peer` events for sensitive actions.

Use **authority mode** for:

- `to_server` events.
- Broadcast events originating from the server.

---

## 🎛 Debugging & Logging

### **7. Enable Debug Mode**
Toggle **Debug** in the EventBridge dock:

- Shows toast notifications for auto-corrections.
- Prints event mapping in the Output console.

---

### **8. Trace Event Usage**
Use **Connections View** in the plugin:

- Displays all subscribers and emitters in the project.
- Double-click opens the script at the exact line of usage.

---

## ✅ Quick Checklist
✔ Use **emit_local** for offline/local logic.  
✔ Use **to_server** for client → server communication.  
✔ Validate **critical actions** on the server.  
✔ Batch frequent updates & use `unreliable` where safe.  
✔ Use **channels** for prioritization.  
✔ Enable **debug mode** during development.  
