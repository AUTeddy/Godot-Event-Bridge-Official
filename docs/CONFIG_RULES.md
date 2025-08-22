# EventBridge Configuration Rules

This document explains **valid configurations**, **invalid combinations**, and **auto-correction logic** for EventBridge events.  
Examples are kept **generic** so they apply to any project.

---

## ✅ Valid Configurations (Recommended)

| **Target**     | **Mode**      | **Sync**       | **Transfer Mode**       | **Channel** | **Notes / Recommended Usage** |
|----------------|---------------|----------------|--------------------------|-------------|-------------------------------|
| **emit_local** | `any_peer`    | `call_local`   | `reliable` *(ignored)*   | 0–4         | Local-only events. `mode` and transfer settings have **no network effect**. |
| **to_server**  | `any_peer` / `authority` | `call_remote`  | `reliable` *(default)*   | 0–4         | Client → Server requests. Use `any_peer` for client requests, `authority` for server loopback. |
| **to_id**      | `authority`   | `call_remote`  | `reliable` *(default)*   | 0–4         | Directed message to a specific peer (server targeting a client). |
| **to_all**     | `authority`   | `call_remote`  | `reliable` *(default)*   | 0–4         | Server broadcasting to everyone. |

> **Tip:** Use `unreliable` / `unreliable_ordered` for **high-frequency, non-critical** updates (e.g., position sync).

---

## 🧭 Examples

```gdscript
# Local-only event
EventManager.ui_button_clicked()

# Client → Server request
EventManager.request_action(player_id, action_type)

# Server → All broadcast
EventManager.round_started(round_number)

# Server → One client
EventManager.private_message(target_peer_id, msg)
```

---

## ❌ Invalid Combinations & Auto‑Correction

| **Invalid Combination**                         | **Why Invalid**                                 | **Auto‑Fix**           |
|-------------------------------------------------|--------------------------------------------------|------------------------|
| `emit_local` + `call_remote`                    | Local events cannot be remote                    | Force `call_local`     |
| `emit_local` + `authority`                      | Authority doesn’t apply to local calls           | Force `any_peer`       |
| `emit_local` + `unreliable(_ordered)`           | Transfer mode irrelevant for local               | Force `reliable`, ch=0 |
| `to_server` + `call_local`                      | Network target requires remote call              | Force `call_remote`    |
| `to_id` / `to_all` + `call_local`               | Network target requires remote call              | Force `call_remote`    |
| `channel < 0` or `channel > 4`                  | Channel out of recommended range                 | Clamp to `0–4`         |

> **Security note:** Treat clients as untrusted. Always validate `to_server` requests on the server.

---

## ⚙️ How EventBridge Applies Config

- During initialization, EventBridge configures RPCs automatically based on event settings.  
- Example:

```gdscript
rpc_config("rpc_event", {
  "rpc_mode": <mode>,
  "call_local": <sync == "call_local">,
  "transfer_mode": <transfer_mode>,
  "channel": <transfer_channel>
})
```

---

## 🔒 Authority & Validation

- **Authority:** Use `authority` for any event that changes game state.  
- **Validation hook:** `EventManager._validate_event(event_name, args)` can block invalid or malicious input.

---

## ✅ Quick Checklist

- Use **`emit_local`** for purely local logic.  
- Use **`to_server`** for client → server requests.  
- Use **`to_all`** / **`to_id`** for server broadcasts and targeted messages.  
- Prefer **`reliable`** for critical game state; use **`unreliable(_ordered)`** for high‑frequency, non‑critical updates.  
- Clamp **channels** to **0–4** unless you configure more in your peer.  
