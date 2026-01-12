## 📄 High-Performance Sharded Log Specification (V13)

### 1. The Core Strategy: Metadata-First, Deferred Block-Commits

The system treats the log as a series of **Atomic Blocks**. While physical byte offsets are calculated sequentially in RAM to update metadata, the actual `.log` disk I/O is deferred until a block is full or a timeout occurs.

### 2. The Active Trigger Mechanism (Ingestion)

Throughput is managed by the **Entry Point** rather than a polling loop:

* **Volume Trigger:** The producer process increments an atomic ETS counter. Once the `@user_stride` is hit (e.g., 1,000 messages), it signals the Shard GenServer via `:force_flush`.
* **Latency Trigger (The Drain):** A 20ms `Process.send_after` timer ensures that if a burst ends at message 1,001, the "remaining 1" is flushed immediately without waiting for a new batch.

### 3. The Iteration & Flush Logic (RAM)

During a flush (triggered by Volume or Time), the Shard GenServer processes the ETS buffer:

* **Step A: Physical Mapping:** Calculate `current_pos = last_pos + size_of(msg_binary)`.
* **Step B: Per-User Bookmarking:** Update the **Bin-Anchor** map for *every* user in the batch.
* **Step C: Stride Detection:** If the message count hits `@user_stride`, generate a **Sparse Index** entry.
* **Step D: The Remainder Sweep:** If the loop reaches the end of the ETS buffer before hitting the next stride, the "leftover" messages are still bundled into the `log_iolist`.

### 4. The Commit Execution (Disk I/O)

To saturate disk bandwidth, writes are batched:

1. **Index Batch:** Write all accumulated index entries to the `.idx` file.
2. **The Log Block:** Perform **one** `:file.write` for the entire `iolist`.
3. **The Anchor Sync:** Update the `Bin-Anchor` with the `final_pos` of the last message in the flush.

---

### 5. Crash Recovery & Resilience

* **The Anchor as Truth:** On startup, the system reads the `Bin-Anchor` to find the `last_physical_offset`.
* **Seek & Resume:** The FD is moved via `:file.position(fd, last_offset)`.
* **Truncation Guard:** Any bytes trailing after the `last_offset` (failed partial writes from a crash) are truncated to ensure a clean, deterministic start.

---

### 6. Implementation Constants

| Constant | Value | Purpose |
| --- | --- | --- |
| `@user_stride` | `1,000` | Block size for Log I/O and Indexing. |
| `@flush_interval` | `20ms` | Max latency for the "Remainder Drain." |
| `@max_buffer` | `5,000` | Hard backpressure limit per shard. |

---

### 🚀 Morning Coding Checklist

1. **[ ] Ingestion Trigger:** Implement `ets.update_counter` in the entry point to send `:force_flush` when `@user_stride` is reached.
2. **[ ] Multi-User Accumulator:** In `perform_flush`, ensure the `Enum.reduce` updates the bookmark map for **every** unique user in the batch.
3. **[ ] Block Write:** Ensure `:file.write(log_fd, iolist)` happens once at the end of the flush, regardless of whether it’s a full 1,000 or a remainder of 1.
4. **[ ] Recovery Seek:** Implement the `init` logic to seek to the `last_physical_offset` found in the Anchor file.

