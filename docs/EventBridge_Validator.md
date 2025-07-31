EventBridge Validator & Networking Best Practices
Overview
EventBridge provides a data-driven event system with multiplayer support for Godot 4. It uses a generated EventManager.gd for easy access to events and relies on an EventBus autoload to dispatch and replicate events across peers.

As of the latest update, EventBridge introduces:

✔ Validation Hooks – Security layer to prevent invalid or malicious events from executing.
✔ Transfer Mode Enhancements – Support for unreliable_ordered for high-performance real-time updates.
✔ Runtime Warnings – Detect unsafe channel configurations at runtime.

Why Validation Exists
In multiplayer games, clients can be compromised or send unexpected data (intentionally or accidentally). If your server trusts all incoming RPC calls blindly, cheating and crashes become possible.

Example risks:

A client sends an invalid dice roll result.

A player triggers a kick_client event to remove others from the game.

A client sends thousands of fake events (spam attack).

Solution: Validators ensure that every incoming event is verified before processing.

How Validators Work
Naming Convention:
For any event my_event, define a function:

```gdscript
func validate_my_event(args: Array) -> bool:
    # Return true to allow, false to block
EventBridge automatically calls this validator (if present) on incoming RPC events before they trigger signals or game logic.
```

Validators run on the receiving peer (usually the server for authority-controlled events).

✅ Example: Dice Game
Event: request_roll

Sent by clients to the server to roll a dice.

Validator Implementation (on server):

```gdscript
func validate_request_roll(args: Array) -> bool:
    # Clients shouldn't send arguments for this event
    if args.size() > 0:
        print("Validator: Invalid arguments for request_roll")
        return false
    return true

#If a hacked client tries:

EventManager.request_roll("give_me_100_points")
# → Blocked by validator, logged as:
```

```gdscript
Validation failed for event 'request_roll'. Ignored.
✅ Example: Secure Chat
Event: send_chat_message

Args: [player_id: int, message: String]
```

Validator Implementation:

```gdscript
func validate_send_chat_message(args: Array) -> bool:
    var player_id = args[0]
    var message = args[1]

    # Ensure the message is short and clean
    if message.length() > 200:
        return false
    if message.begins_with("/kick"):
        return false # Prevent abuse of chat commands
    return true
```
    
Where Validators Live
Add them in EventManager.gd (the generated file) or in a script that extends it.

They must not modify the args; they only approve or reject.

Transfer Modes Explained
EventBridge supports three transfer modes:

reliable – Guaranteed delivery, ordered (default).

unreliable – Faster, but packets may be lost (use for non-critical updates).

unreliable_ordered – Fast and ordered, but older packets may be dropped if newer arrive.

✅ When to Use unreliable_ordered
Example: Player position updates in a fast-paced game.

You only care about the latest position, not old ones.

If packet #1 arrives after #2, it is discarded automatically.

Reduces bandwidth and avoids "rubberbanding".

Configuration in Event Dock:

Set Transfer Mode = unreliable_ordered.

Assign a dedicated channel (avoid sharing with other events).

```gdscript
Channel Safety
Godot supports multiple ENet channels for parallel streams of events.
Rule:
When using unreliable_ordered, do NOT mix different event types on the same channel, or packets for one event may drop packets for another.
```

EventBridge helps you by:

UI Warning: In the Event Dock if multiple events share the same channel.

Runtime Warning: Logs conflicts during execution.

✅ Bad Example (Will cause dropped packets):
```gdscript
Channel 0 → player_position_update (unreliable_ordered)
Channel 0 → enemy_spawn (reliable)
If enemy_spawn arrives after a position update, it might block future updates.
```

✅ Good Example:
```gdscript
Channel 0 → player_position_update (unreliable_ordered)
Channel 1 → enemy_spawn (reliable)
Developer Workflow Summary
Create Events in Event Dock.
```

Assign transfer modes:

reliable for critical game logic (dice rolls, inventory updates).

unreliable or unreliable_ordered for high-frequency data (movement, effects).

Add validators for authority-side events (e.g., to_server).

Monitor warnings in:

Dock Toasts: for config mistakes.

Console Logs: for runtime issues.

Quick Validator Cheat Sheet
Use Case	Example Event	Validator Example
Block invalid args	request_roll	Reject if args not empty
Anti-cheat checks	attack_enemy	Reject if enemy_id is invalid
Limit spam	send_chat_message	Reject if too many messages/sec
Security enforcement	kick_client	Reject if caller is not admin

## ✅ With validators + proper transfer mode configuration, EventBridge now provides both performance and security for multiplayer Godot games.


## ✅ 1. Real-World Validator Implementation (validators.gd)
Create this as an autoload or attach it to your main game node.

```gdscript

# File: res://scripts/validators.gd
extends Node

# === Security & Anti-Cheat Validators for EventBridge ===
# These are examples you can modify for your game.

# 1. Prevent invalid dice roll requests
func validate_request_roll(args: Array) -> bool:
    # Clients should not send any arguments for this event
    if args.size() > 0:
        print("[Validator] request_roll rejected: unexpected args")
        return false
    return true

# 2. Prevent invalid dice results (server sending to clients)
func validate_dice_result(args: Array) -> bool:
    var result = args[0] if args.size() > 0 else null
    # Only allow numbers between 1 and 6
    if result == null or result < 1 or result > 6:
        print("[Validator] dice_result rejected: invalid value")
        return false
    return true

# 3. Prevent unauthorized kick
func validate_kick_client(args: Array) -> bool:
    var msg = args[0]
    # Only allow this event from server authority
    if !get_tree().is_network_server():
        print("[Validator] kick_client rejected: unauthorized peer")
        return false
    return true

# 4. Secure chat system
func validate_send_chat_message(args: Array) -> bool:
    if args.size() != 2:
        return false
    var message = args[1]
    if message.length() > 200:
        return false
    if message.begins_with("/kick"):
        return false # Disallow dangerous commands in chat
    return true

# 5. Rate limit example (anti-spam)
var last_message_time := {}
const CHAT_COOLDOWN = 1.0 # seconds
func validate_chat_spam(args: Array) -> bool:
    var player_id = str(args[0])
    var now = Time.get_unix_time_from_system()
    if last_message_time.has(player_id) and now - last_message_time[player_id] < CHAT_COOLDOWN:
        print("[Validator] Player %s spam detected" % player_id)
        return false
    last_message_time[player_id] = now
    return true
```

✅ How It Works:
Each function follows the auto-naming pattern:
validate_<event_name>(args: Array) -> bool

Return true to allow, false to block.

Print messages for debugging (can later replace with logs or telemetry).

✅ 2. Event Flow Diagram
Here’s a clear diagram of how validation fits into the event system:

```
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

Key Points
Validation only happens on the receiving side (usually the server for authority).

If a validator exists and returns false, the event is ignored.

If no validator exists, the event is allowed by default.

✅ 3. Developer Best Practices
Always add validators for:

Events targeting the server (to_server).

Events that modify critical game state (e.g., score updates, item trades).

Use rate limiting for spam-prone events like chat or emotes.

Keep validators lightweight—they run in the network loop.

Combine validators with server authority checks for extra security.
