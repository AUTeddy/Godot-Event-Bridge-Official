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

gdscript
Copy
Edit
func validate_my_event(args: Array) -> bool:
    # Return true to allow, false to block
EventBridge automatically calls this validator (if present) on incoming RPC events before they trigger signals or game logic.

Validators run on the receiving peer (usually the server for authority-controlled events).

✅ Example: Dice Game
Event: request_roll

Sent by clients to the server to roll a dice.

Validator Implementation (on server):

gdscript
Copy
Edit
func validate_request_roll(args: Array) -> bool:
    # Clients shouldn't send arguments for this event
    if args.size() > 0:
        print("Validator: Invalid arguments for request_roll")
        return false
    return true
If a hacked client tries:

gdscript
Copy
Edit
EventManager.request_roll("give_me_100_points")
→ Blocked by validator, logged as:

csharp
Copy
Edit
Validation failed for event 'request_roll'. Ignored.
✅ Example: Secure Chat
Event: send_chat_message

Args: [player_id: int, message: String]

Validator Implementation:

gdscript
Copy
Edit
func validate_send_chat_message(args: Array) -> bool:
    var player_id = args[0]
    var message = args[1]

    # Ensure the message is short and clean
    if message.length() > 200:
        return false
    if message.begins_with("/kick"):
        return false # Prevent abuse of chat commands
    return true
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

Channel Safety
Godot supports multiple ENet channels for parallel streams of events.
Rule:
When using unreliable_ordered, do NOT mix different event types on the same channel, or packets for one event may drop packets for another.

EventBridge helps you by:

UI Warning: In the Event Dock if multiple events share the same channel.

Runtime Warning: Logs conflicts during execution.

✅ Bad Example (Will cause dropped packets):
scss
Copy
Edit
Channel 0 → player_position_update (unreliable_ordered)
Channel 0 → enemy_spawn (reliable)
If enemy_spawn arrives after a position update, it might block future updates.

✅ Good Example:
scss
Copy
Edit
Channel 0 → player_position_update (unreliable_ordered)
Channel 1 → enemy_spawn (reliable)
Developer Workflow Summary
Create Events in Event Dock.

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

✅ With validators + proper transfer mode configuration, EventBridge now provides both performance and security for multiplayer Godot games.
