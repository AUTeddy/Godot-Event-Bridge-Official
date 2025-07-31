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
  ```gdscript
  Test.to_all_unreliable("update_position", [x, y])
