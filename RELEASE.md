I’ve integrated your observations—specifically the **Kafka-style partitioning**, the **Backpressure/Throttle** mechanics, and the **Page-Alignment** considerations—into the final architectural specification.

Here is the updated "Master Blueprint" for your implementation tomorrow morning.

---

## 🏗️ The BimipLog High-Throughput Engine (V11)

**Design Goal:** 1,000,000 msg/sec via Elixir-native Sharded WAL.

### 1. Ingestion & Throttle (Parallel)

* **Binary Pre-Packaging:** The caller (client) performs `term_to_binary` and header assembly. The Shard Actor receives a "Ready-to-Write" blob.
* **Producer Backpressure:** If the ETS buffer size exceeds `@max_buffer_per_shard`, the system returns `{:error, :backpressure}`. This prevents RAM exhaustion during disk I/O spikes.
* **Logical Ordering:** Parallel workers fetch the next monotonic offset via `ets.update_counter`.

### 2. The "Triple-Action" Batch Engine (Sequential)

Every `flush_interval` (20ms) or when a `stride` is hit, the Shard GenServer performs a **Single-Pass Transformation**:

1. **Head (Sparse Index):** Identifies messages hitting the `@user_stride`. Captures their `current_pos` and builds an index entry.
2. **Body (The Log):** Accumulates all message binaries into one massive `iolist`. Calculates future byte-offsets sequentially in RAM.
3. **Caboose (The Anchor):** Captures the metadata and physical byte-offset of the **very last** message in the batch.

### 3. The Big Bang Commit (Surgical Strike)

To saturate NVMe bandwidth and minimize syscalls, the engine performs:

* **Log Write:** One big push of the `iolist` body to the `.log` file.
* **Index Write:** One push of the accumulated index entries to the `.idx` file.
* **Anchor Update:** An atomic `:file.pwrite` of the "Caboose" data to the `Bin-Anchor` file.

---

## 🛠️ Performance & Durability Policy

* **Page Alignment:** We aim for batch sizes that are multiples of 4KB to align with SSD hardware pages, reducing "Write Amplification."
* **Isolation:** Each shard owns its own File Descriptors (FDs). There is **zero lock contention** between Shard 0 and Shard 63.
* **Fsync Policy:** Following the Kafka model, we rely on the OS Page Cache for performance, but the **Bin-Anchor** ensures that even in a crash, we know exactly where the "Known Good" data ends.

---

## 🏁 Morning Implementation Steps

1. **Refactor `perform_flush`:** Remove all `:file.write` calls from inside the loop.
2. **Accumulator:** Use `Enum.reduce` to return `{log_batch, index_batch, last_item, final_pos}`.
3. **Sequential Write:** Call `:file.write` twice (Log, then Index) and `:file.pwrite` once (Anchor).
4. **Recovery:** Update `init` to seek to the `last_physical_offset` found in the Anchor.

This design is now rock solid. You have the speed of a batch-writer with the precision of a single-record log.

**I'm ready when you are. Would you like me to hold this blueprint for you to reference when you start coding in the morning?**
