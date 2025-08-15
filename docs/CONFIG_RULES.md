# EventBridge Configuration Rules (Project‑Aware)

This document covers **valid configurations**, **invalid combinations**, and **auto-correction logic** for EventBridge events.  
It also reflects the **current state of your project** by reading from `event_registry.json` and `event_bus.gd`.

---

## 📦 Project Snapshot

- **Namespaces & event counts:** Admin (7), Clients (1), Game (4), Lobby (2), Server (2)
- **RPC handler signature in code:**  
  ```gdscript
    @rpc("any_peer", "call_local", "reliable")
        func rpc_event(
  ```

*Meaning:* Incoming RPCs are allowed from `any_peer`, executed **locally** (`call_local`) and default to **reliable** unless overridden at runtime via `rpc_config(...)`.

---

## ✅ Valid Configurations (Recommended)

| **Target**     | **Mode**      | **Sync**       | **Transfer Mode**       | **Channel** | **Notes / Recommended Usage** |
|----------------|---------------|----------------|--------------------------|-------------|-------------------------------|
| **emit_local** | `any_peer`    | `call_local`   | `reliable` *(ignored)*   | 0–4         | Local-only events. `mode` and transfer settings have **no network effect**. |
| **to_server**  | `authority`   | `call_remote`  | `reliable` *(default)*   | 0–4         | Client → Server requests. Server validates and acts. |
| **to_id**      | `any_peer`    | `call_remote`  | `reliable` *(default)*   | 0–4         | Directed message to a specific peer (usually from the server). |
| **to_all**     | `any_peer`    | `call_remote`  | `reliable` *(default)*   | 0–4         | Server (authoritative source) broadcasting to everyone. |

> **Tip:** Use `unreliable` / `unreliable_ordered` for **high-frequency, non-critical** updates (e.g., cosmetic animation sync).

---

## 🧭 Examples from Your Registry

Below are a few real entries parsed from your `event_registry.json`:

| **Event** | **Target** | **Mode** | **Sync** | **Transfer** | **Ch.** | **Args** |
|---|---|---|---|---|---|---|
| `Admin::client_connected` | `to_all` | `any_peer` | `call_remote` | `reliable` | `0` | player_id:int |
| `Admin::client_disconnected` | `to_id` | `authority` | `call_remote` | `reliable` | `0` | player_id:int |
| `Clients::press_key_h` | `to_server` | `authority` | `call_remote` | `reliable` | `1` | player_id:int |
| `Game::dice_result` | `to_id` | `any_peer` | `call_remote` | `reliable` | `0` | result:int |
| `Game::dice_broadcast` | `to_all` | `any_peer` | `call_remote` | `reliable` | `0` | player_id:int, result:int |
| `Lobby::ping` | `to_server` | `authority` | `call_remote` | `reliable` | `0` | — |
| `Lobby::pong` | `emit_local` | `any_peer` | `call_local` | `reliable` | `0` | msg:String |
| `Server::add_to_log` | `to_server` | `authority` | `call_remote` | `reliable` | `0` | sender:String, msg:String, level:int |
| `Server::welcome_to_client` | `to_id` | `authority` | `call_remote` | `reliable` | `0` | msg:String |

> Ensure your multiplayer peer supports the number of channels you configure (e.g., ENet with `set_channel_count(5)`).

---

## ❌ Invalid Combinations & Auto‑Correction

| **Invalid Combination**                         | **Why Invalid**                                 | **Auto‑Fix**           |
|-------------------------------------------------|--------------------------------------------------|------------------------|
| `emit_local` + `call_remote`                    | Local events cannot be remote                    | Force `call_local`     |
| `emit_local` + `authority`                      | Authority doesn’t apply to local calls           | Force `any_peer`       |
| `emit_local` + `unreliable(_ordered)`           | Transfer mode/channel irrelevant for local       | Force `reliable`, ch=0 |
| `to_server` + `any_peer`                        | Server RPCs must be authority-gated              | Force `authority`      |
| `to_server` + `call_local`                      | Network target requires remote call              | Force `call_remote`    |
| `to_id` / `to_all` + `call_local`               | Network target requires remote call              | Force `call_remote`    |
| `channel < 0` or `channel > 4`                  | Channel out of recommended range                 | Clamp to `0–4`         |

> **Security note:** Treat clients as untrusted. Validate all `to_server` requests server-side.

---

## ⚙️ How EventBridge Applies Config (Current Implementation)

- **Where it’s set:** During initialization, the plugin iterates events and calls:
  ```gdscript
  rpc_config("rpc_event", {
    "rpc_mode": <from `mode`>,
    "call_local": <from `sync` == "call_local">,
    "transfer_mode": <from `transfer_mode`>,
    "channel": <from `transfer_channel`>
  })
  ```
---

## 🔒 Authority & Validation

- **Authority:** Use `authority` for any event that changes game state.  
- **Validation hook:** `EventManager._validate_event(event_name, args)` is called before dispatch. Implement per‑event validators (e.g., `validate_move(args)`) to block invalid or malicious input.


---

## ✅ Quick Checklist

- Use **`emit_local`** for purely local logic (no network effects).
- Use **`to_server`** for client → server requests and validate on the server.
- Use **`to_all`** / **`to_id`** for server‑side broadcasts and directed messages.
- Prefer **`reliable`** for critical game state; use **`unreliable(_ordered)`** for high‑frequency, non‑critical updates.
- Keep **channels** organized by priority; clamp to **0–4** unless you know you need more.
- Ensure **per‑event config is applied at send time** to avoid the “last config wins” pitfall.

---
