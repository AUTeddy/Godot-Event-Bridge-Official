
# EventBridge Validator & Networking — Developer Guide

## 📚 Overview

```text
Client (EventManager API) ──► EventBus Autoload ──► RPC System ──► EventBus (Receiving Peer)
                                                                  │
                                                                  ▼
                                                          EventManager._validate_event()
                                                                  │
                                    ┌─────────────────────────────┴────────────────────────────┐
                                    │ YES (Valid)                                             │ NO (Invalid)
                                    ▼                                                        ▼
                            emit_signal(event_name, args)                        Block event, log warning
```



## ✨ What’s New (Latest Update)

- ✔ **Validation Hooks** – Security layer to block invalid or malicious events before they reach gameplay.
- ✔ **Transfer Mode Enhancements** – `unreliable_ordered` for smooth, high‑frequency updates.
- ✔ **Runtime Warnings** – Detect unsafe channel configurations during playtests.

---

## 🔒 Why Validation Exists

In multiplayer, clients can be compromised or simply buggy. If the server trusts all RPCs, you risk cheats and crashes.

**Example risks:**

- A client sends an **invalid dice roll** result.
- A player triggers a **kick_client** event to remove others.
- A client spams **thousands of events** to degrade performance.

**Solution:** **Validators** verify incoming events **before** any game logic runs.

---

## 🧠 How Validators Work

### Naming Convention

For any event `my_event`, define a function:

```gdscript
func validate_my_event(args: Array) -> bool:
    # Return true to allow, false to block
    return true
```

EventBridge automatically calls this validator (if present) on the **receiving peer** *before* emitting signals or invoking callbacks.

### Where Validation Happens

1. A peer sends an event via the generated API, e.g.:
   ```gdscript
   EventManager.some_event(arg1, arg2)
   ```
2. This calls an internal `_invoke_event()` in the EventBus (autoload).
3. The bus decides the path: **local**, **to_server**, **to_all**, or **to_id**.
4. For networked events, it uses `rpc()` / `rpc_id()`.
5. The receiving peer handles it in the RPC handler:
   ```gdscript
   @rpc func rpc_event(ns_name: String, event_name: String, args: Array) -> void:
       # Validation check happens here (see below)
   ```

> _Note:_ If your actual autoload name differs (e.g., `EventBusAutoload`), adjust accordingly.

---

## ✅ Validator Check (in `rpc_event`)

Inside `event_bus.gd` you should perform the check before emitting:

```gdscript
# Pseudocode/sketch — adapt to your project structure/names
@rpc("any_peer", "call_local", "reliable")
func rpc_event(ns_name: String, event_name: String, args: Array) -> void:
    # If EventManager is an autoload, it's accessible as a global
    # singleton by the name you configured in Project Settings.
    # Example: if you autoloaded it as "EventManager", you can use it directly.
    var ok := true
    if "EventManager" in ProjectSettings.get_setting("autoload", {}):
        ok = EventManager._validate_event(event_name, args)
    elif Engine.has_singleton("EventManager"):
        # Only if you registered it as an engine singleton (rare)
        var em = Engine.get_singleton("EventManager")
        ok = em._validate_event(event_name, args)

    if not ok:
        if DEBUG:
            EventBridgeLogger.event_log("EventBus", "Blocked by validator: %s::%s" % [ns_name, event_name], 2)
        return

    # Emit only after validation
    var signal_name := ns_name + "__" + event_name
    if has_signal(signal_name):
        # Forward args: callv is used to splat the array after the signal name
        callv("emit_signal", [signal_name] + args)
```

> If you use a different mechanism to access the EventManager (e.g., `get_node("/root/EventManager")`), adapt accordingly.

---

## 🔍 How `_validate_event()` Works (Generated)

In the generated `EventManager.gd`, a helper typically:

```gdscript
func _validate_event(event_name: String, args: Array) -> bool:
    var validator_name := "validate_" + event_name
    if has_method(validator_name):
        var ok := call(validator_name, args)
        if not ok:
            push_warning("Validation failed for event '%s'. Ignored." % event_name)
            return false
    return true
```

**Meaning:** If you implement a method named `validate_<event>` anywhere that **extends** or **replaces** the EventManager autoload, it will be picked up and executed.

---

## 🧪 Examples

### 1) Secure Dice Roll

**Event:** `request_roll` (client → server)

**Validator:** reject unexpected args.
```gdscript
func validate_request_roll(args: Array) -> bool:
    # Clients shouldn't send any arguments
    if args.size() > 0:
        print("Validator: Invalid args for request_roll")
        return false
    return true
```

**Hacked client tries:**
```gdscript
EventManager.request_roll("give_me_100_points")
# -> Blocked by validator. No signal emitted.
```

---

### 2) Secure Chat

**Event:** `send_chat_message(args: [player_id: int, message: String])`

**Validator:**
```gdscript
func validate_send_chat_message(args: Array) -> bool:
    if args.size() != 2:
        return false
    var player_id = args[0]
    var message: String = args[1]
    if message.length() > 200:
        return false
    if message.begins_with("/kick"):
        return false # Prevent abuse of chat commands
    return true
```

---

### 3) Authority Gate (Kick)

**Event:** `kick_client(args: [target_id: int])`

**Validator:**
```gdscript
func validate_kick_client(args: Array) -> bool:
    # Only allow from server authority
    return get_tree().is_network_server()
```

---

### 4) Simple Rate Limiter (Anti‑Spam)

```gdscript
var _last_message_time := {}
const CHAT_COOLDOWN := 1.0 # seconds

func validate_chat_message_rate(args: Array) -> bool:
    var player_id := str(args[0])
    var now := Time.get_unix_time_from_system()
    if _last_message_time.has(player_id) and now - _last_message_time[player_id] < CHAT_COOLDOWN:
        print("[Validator] Spam detected for player %s" % player_id)
        return false
    _last_message_time[player_id] = now
    return true
```

> Keep validators **lightweight**; they run in the networking path.

---

## 🗂️ Where Validators Live

You have two options:

### Option A: Inline (small projects)
Add the functions directly into the generated `EventManager.gd` **(below the autogenerated region)** or into a partial/extension if your setup supports it.

```gdscript
func validate_request_roll(args: Array) -> bool:
    return args.size() == 0
```

### Option B: Separate File (recommended for larger projects)
Create `res://autoload/event_validators.gd` and **extend** the EventManager. Autoload this script **instead of** (or **in addition to**) the generated one, depending on your architecture.

```gdscript
# File: res://autoload/event_validators.gd
extends EventManager

func validate_request_roll(args: Array) -> bool:
    return args.size() == 0

func validate_kick_client(args: Array) -> bool:
    return get_tree().is_network_server()
```

> Because it **extends** `EventManager`, `has_method("validate_x")` will return true on the autoload instance, and your custom logic will run.

---

## 🚦 Transfer Modes & Channel Safety

EventBridge supports three transfer modes:

- **reliable** – Guaranteed, ordered (default). Use for critical state (turns, inventory).
- **unreliable** – Best effort. Use for non‑critical, high‑frequency updates (FX).
- **unreliable_ordered** – Ordered per **channel**, drops outdated packets. Ideal for positions/aim.

### When to use `unreliable_ordered`
- You only care about the **latest** state (e.g., positions).
- Older packets can be dropped without harm (prevents rubberbanding).

**EventDock Configuration:**  
- Set **Transfer Mode** = `unreliable_ordered`  
- Assign a **dedicated channel** (don’t mix with other event types).

```gdscript
# Channel Safety Rule
# Do NOT mix different event types on the same channel when using unreliable_ordered,
# or packets for one may cause others to drop.
```

**Bad Example (causes drops/latency):**
```text
Channel 0 → player_position_update (unreliable_ordered)
Channel 0 → enemy_spawn            (reliable)
```

**Good Example:**
```text
Channel 0 → player_position_update (unreliable_ordered)
Channel 1 → enemy_spawn            (reliable)
```

**EventBridge helps you:**
- **Dock warnings** for shared channels with risky modes.
- **Runtime warnings** when conflicts are detected.

---

## 🧪 Testing Validators

1. Enable **Debug** in the EventDock (toasts + console mapping).
2. Write a failing validator (e.g., reject messages > 200 chars).
3. Send an oversized message from a client.
4. Confirm the event is **blocked** and the log shows a warning.
5. Verify **no gameplay system** reacted to the blocked event.

---

## ✅ Quick Validator Cheat Sheet

| Use Case                 | Example Event        | Validator Strategy                    |
|-------------------------|----------------------|---------------------------------------|
| Block invalid args      | `request_roll`       | Reject if `args.size() != 0`          |
| Anti‑cheat checks       | `attack_enemy`       | Reject if `enemy_id` invalid          |
| Spam limiting           | `send_chat_message`  | Per‑peer cooldown                     |
| Security enforcement    | `kick_client`        | Allow only if server authority        |

---

## 🔧 Developer Workflow Summary

1. **Create events** in EventDock.  
2. Assign **transfer modes** (`reliable` vs `unreliable(_ordered)`) and **channels**.  
3. Implement **validators** for authority‑side and critical events.  
4. **Regenerate** if you changed registry or API.  
5. **Test** with Debug enabled and watch warnings/logs.

---

**With validators + proper transfer configuration, EventBridge delivers both performance _and_ security for multiplayer Godot games.**
