# EventBridge Configuration Rules

## ✅ Valid Configurations (Recommended Settings)

| **Target**       | **Mode**        | **Sync**        | **Transfer Mode**      | **Channel** | **Notes / Recommended Usage**                                   |
|------------------|---------------|-----------------|-------------------------|-------------|----------------------------------------------------------------|
| **emit_local**   | `any_peer`    | `call_local`    | `reliable` *(default)* | 0–4         | Local-only events. `Mode` irrelevant. Transfer settings have no effect. |
| **to_server**    | `authority`   | `call_remote`   | `reliable` *(default)* | 0–4         | Client → Server calls. Requires `authority` mode.             |
| **to_id**        | `any_peer`    | `call_remote`   | `reliable` *(default)* | 0–4         | Peer → Peer messaging. Usually controlled by server.          |
| **to_all**       | `any_peer`    | `call_remote`   | `reliable` *(default)* | 0–4         | Broadcast from server (or authoritative peer).                |

---

## ❌ Invalid Combos & Auto-Correction Behavior

| **Invalid Combination**                                      | **Why Invalid**                                      | **Auto-Corrected To**                                    |
|--------------------------------------------------------------|------------------------------------------------------|---------------------------------------------------------|
| `emit_local` + `call_remote`                                | Local events cannot use remote calls                | `call_local`                                           |
| `emit_local` + `authority`                                  | Mode irrelevant for local events                    | `any_peer`                                             |
| `emit_local` + `unreliable` / `unreliable_ordered`          | Transfer mode irrelevant for local events           | `reliable`                                             |
| `to_server` + `any_peer`                                    | Server calls must use authority mode                | `authority`                                            |
| `to_server` + `call_local`                                  | Network targets require remote calls                | `call_remote`                                          |
| `to_id` / `to_all` + `call_local`                           | Network targets require remote calls                | `call_remote`                                          |
| `transfer_channel < 0` or `> 4`                             | Invalid channel range                               | Clamped to `0–4`                                       |

---

### 🔍 Auto-Correction Logic Flow

![Auto-Correction Flowchart](docs/images/eventbridge_auto_correction_flowchart.png)

*(SVG for docs: `docs/images/eventbridge_auto_correction_flowchart.svg`)*

---

### 🔄 Invalid → Auto-Fix Mapping

![Invalid → Auto-Fix Diagram](docs/images/eventbridge_invalid_autofix_mapping.png)

*(SVG for docs: `docs/images/eventbridge_invalid_autofix_mapping.svg`)*
