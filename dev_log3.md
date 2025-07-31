✅ Auto-Correction Flowchart (Logic Overview)
Description
This flow shows how EventBridge decides corrections for each Target.

[Start]
   |
   v
Check Target:
   |
   |-- emit_local --------------------------+
   |                                        |
   |    Sync != call_local  →  fix to call_local
   |    Mode != any_peer    →  fix to any_peer
   |    Transfer != reliable → fix to reliable
   |    Transfer = unreliable_ordered → reliable
   |                                        |
   +----------------------------------------+
   |
   |-- to_server ---------------------------+
   |                                        |
   |    Mode != authority  →  fix to authority
   |    Sync != call_remote → fix to call_remote
   |                                        |
   +----------------------------------------+
   |
   |-- to_id / to_all ----------------------+
        |
        |    Sync != call_remote → fix to call_remote
        |
   +----------------------------------------+
   |
Check Transfer Channel:
   |
   |    channel < 0 → 0
   |    channel > 4 → 4
   |
[End]

✅ Dropdown Tooltips (Recommended UI)
Here’s a set of tooltips for each dropdown option to guide users in the editor:

Target Dropdown Tooltips
emit_local → “Dispatches event locally only. Ignores network settings.”

to_server → “Sends event to the server. Requires authority mode.”

to_id → “Sends event to a specific peer by ID.”

to_all → “Broadcasts event to all connected peers.”

Mode Dropdown Tooltips
authority → “Event can only be emitted by the server or the authority peer.”

any_peer → “Event can be emitted by any connected peer.”

Sync Dropdown Tooltips
call_remote → “Executes the event remotely via RPC.”

call_local → “Executes the event locally (used for emit_local only).”

Transfer Mode Dropdown Tooltips
reliable → “Guaranteed delivery in order. Use for important events (default).”

unreliable → “Faster, but packets can be lost. Use for non-critical updates.”

unreliable_ordered → “Packets can be dropped but order is preserved. Use for high-frequency updates.”

Transfer Channel Tooltip
0–4 → “Channels separate traffic priority. Lower channels typically have higher priority.”

✅ You can apply these tooltips in Godot using:

gdscript
Kopieren
Bearbeiten
rpc_targets.set_item_tooltip(index, "Tooltip text here")
