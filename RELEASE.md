## 📄 High-Performance Sharded Log Specification (V12)

### 1. The Core Strategy: Metadata-First, Block-Log Last

Instead of writing a message to the log as soon as its position is known, we **defer the log I/O** until a full logical block (Stride) is completed. However, we update the **Bookmark (Anchor)** and **Sparse Index** during the calculation to maintain a real-time map of the storage.

### 2. The Iteration Logic (RAM)

During a flush event (triggered by timer or buffer size), the Shard GenServer processes the messages in a single sequential pass:

* **Step A: Physical Mapping:** For every message in the batch, calculate `current_pos = last_pos + size_of(msg_binary)`.
* **Step B: Per-User Bookmarking:** Immediately update the **Bin-Anchor** map for that specific user. This ensures that even if User 1 is at the start of a 1,000-message batch, their progress is recorded.
* **Step C: Stride Detection:** If the message hits the `@user_stride` (e.g., every 5th or 100th message), generate an entry for the **Sparse Index Batch**.
* **Step D: Log Buffering:** Append the binary to a RAM-resident `iolist`.

### 3. The Commit Execution (Disk I/O)

Disk writes are only triggered when the **Stride is Full** or the **Flush Timer Expires**.

1. **Index Batch:** Write the accumulated index entries to the `.idx` file.
2. **The Log Block:** Perform **one** `:file.write` for the entire `iolist` accumulated during the stride.
3. **The Anchor Sync:** Atomic update to the `Bin-Anchor` file to "seal" the physical offset.

---

### 4. Crash Recovery & Persistence

The system is designed to be **self-healing** without requiring a full log scan.

* **The Anchor as Truth:** On startup, the GenServer reads the `Bin-Anchor`. It retrieves the `last_physical_offset` that was successfully committed.
* **Seek & Resume:** The file descriptor is moved to that exact byte via `:file.position(fd, last_offset)`.
* **Truncation Guard:** Any data existing in the file *after* the `last_offset` (remnants of a partial write during a crash) is ignored or truncated. This ensures the log starts from a known-good, deterministic state.

---

### 5. Implementation Constants (Suggested)

| Constant | Value | Purpose |
| --- | --- | --- |
| `@user_stride` | `5` to `100` | Balancing lookup speed vs. index file size. |
| `@flush_interval` | `20ms` | Maximum latency a message waits before being forced to disk. |
| `@max_batch_size` | `1,000` | Maximum messages to group into a single Log Block. |

---

### 🚀 Morning Coding Checklist

1. **[ ]** Refactor `perform_flush` to use an `Enum.reduce` that builds `log_iolist` and `index_iolist` while updating a user-map.
2. **[ ]** Ensure the `:file.write` for the log happens **outside** the loop, only when the stride/batch is complete.
3. **[ ]** Implement the `init` recovery that reads the `Bin-Anchor` and performs the initial `:file.position`.
4. **[ ]** Verify that the `Bin-Anchor` is updated with the **very last** calculated offset of the batch.

**Documentation complete. The architecture is locked. Would you like me to generate a skeleton Elixir module based on this documentation to give you a head start in the morning?**
