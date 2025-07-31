# 📘 EventBridge Developer Guide

A complete reference for configuring, validating, and optimizing EventBridge events in Godot. Includes **valid configuration tables**, **auto-correction rules**, **flow diagrams**, and **advanced tips**.

---

## ✅ Valid Event Configurations

| **Target**       | **Mode**        | **Sync**        | **Transfer Mode**      | **Channel** | **Notes / Recommended Usage**                                   |
|------------------|---------------|-----------------|-------------------------|-------------|----------------------------------------------------------------|
| **emit_local**   | `any_peer`    | `call_local`    | `reliable` *(default)* | 0–4         | Local-only events. `Mode` irrelevant. Transfer settings have no effect. |
| **to_server**    | `authority`   | `call_remote`   | `reliable` *(default)* | 0–4         | Client → Server calls. Requires `authority` mode.             |
| **to_id**        | `any_peer`    | `call_remote`   | `reliable` *(default)* | 0–4         | Peer → Peer messaging. Usually controlled by server.          |
| **to_all**       | `any_peer`    | `call_remote`   | `reliable` *(default)* | 0–4         | Broadcast from server (or authoritative peer).                |

---

## ❌ Invalid Combos & Auto-Correction

| **Invalid Combination**                                      | **Reason**                                      | **Auto-Corrected To**                                    |
|--------------------------------------------------------------|-------------------------------------------------|---------------------------------------------------------|
| `emit_local` + `call_remote`                                | Local events cannot use remote calls           | `call_local`                                           |
| `emit_local` + `authority`                                  | Mode irrelevant for local events               | `any_peer`                                             |
| `emit_local` + `unreliable` / `unreliable_ordered`          | Transfer mode irrelevant for local events      | `reliable`                                             |
| `to_server` + `any_peer`                                    | Server calls must use authority mode           | `authority`                                            |
| `to_server` + `call_local`                                  | Network targets require remote calls           | `call_remote`                                          |
| `to_id` / `to_all` + `call_local`                           | Network targets require remote calls           | `call_remote`                                          |
| `transfer_channel < 0` or `> 4`                             | Invalid channel range                          | Clamped to `0–4`                                       |

---

## 🔍 Auto-Correction Logic Flow

![Auto-Correction Flowchart](images/eventbridge_auto_correction_flowchart.png)

*(SVG version: [Download](images/eventbridge_auto_correction_flowchart.svg))*

---

## 🔄 Invalid → Auto-Fix Mapping

![Invalid → Auto-Fix Diagram](images/eventbridge_invalid_autofix_mapping.png)

*(SVG version: [Download](images/eventbridge_invalid_autofix_mapping.svg))*

---

## 🌐 Networking Best Practices

### **3. Transfer Channels**
Prioritize traffic using channels:
- **Channel 0** → Highest priority (critical events).
- **Channel 1–4** → Secondary traffic (UI sync, animations).
- Separate channels reduce packet queuing delays.

---

### **4. Use ENet’s Features**
Godot’s `NetworkedMultiplayerENet` supports:
- **Compression** → Reduce bandwidth usage.
- **Packet Merging** → Enable `enet_compress` in Project Settings.
- Use `peer.set_channel_count(5)` for more than 4 channels.

---

## 🔐 Security & Authority Control

### **5. Authority Mode**
- Only the server should trigger critical events (e.g., combat results, turn change).
- Clients should only emit `to_server` calls.
- Validate all client events on the server before applying game logic.

---

### **6. Prevent RPC Injection**
- Never trust `any_peer` for sensitive actions.
- Use **authority** for:
  - `to_server` events.
  - Broadcasts from the server.

---

## 🎛 Debugging & Logging

### **7. Enable Debug Mode**
- Toggle **Debug** in EventBridge dock.
- Displays toast notifications for auto-corrections.
- Prints event mapping to the Output console.

---

### **8. Trace Event Usage**
- Use **Connections View** in the EventBridge plugin.
- Shows all subscribers and emitters in your project.
- Double-click to jump to the script and line of usage.

---

## ✅ Quick Checklist
✔ Use **emit_local** for offline/local logic.  
✔ Use **to_server** for client → server communication.  
✔ Validate **critical actions** on the server.  
✔ Batch frequent updates & use `unreliable` where safe.  
✔ Use **channels** for prioritization.  
✔ Enable **debug mode** during development.  

---

### 🔗 **Additional Docs**
- [Event Auto-Correction Rules](CONFIG_RULES.md)
- [Advanced Performance Tips](ADVANCED_TIPS.md)
