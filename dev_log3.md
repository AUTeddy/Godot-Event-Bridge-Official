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
