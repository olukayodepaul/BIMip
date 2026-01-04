Got it! Here's a **reduced, concise implementation doc** for `fetch_for_device/4`:

---

## **Function:** `Queue.QueueLogImpl.fetch_for_device/4`

### **Purpose**

Fetch a single message for a user and partition **skipping messages sent by the same device**.
Checks **ETS buffer first**, then **disk via index cache**.

---

### **Parameters**

| Param          | Example          | Description                                          |
| -------------- | ---------------- | ---------------------------------------------------- |
| `user`         | `"a@domain.com"` | User whose messages are fetched                      |
| `partition_id` | `1`              | Partition/shard ID                                   |
| `device_id`    | `1`              | Device fetching messages (used to skip own messages) |
| `offset`       | `1`              | Message offset in the log                            |

---

### **Return Values**

| Condition                                | Return                      |
| ---------------------------------------- | --------------------------- |
| Message exists & device ≠ writer_device  | `{:ok, [%Bimip.Message{}]}` |
| Message exists & device == writer_device | `{:ok, []}`                 |
| Message not found                        | `{:error, :not_found}`      |
| Disk read failed                         | `{:error, :read_failed}`    |

---

### **Examples**

```elixir
# Fetch own message → skipped
Queue.QueueLogImpl.fetch_for_device("a@domain.com", 1, 1, 1)
# => {:ok, []}

# Fetch from another device → returned
Queue.QueueLogImpl.fetch_for_device("a@domain.com", 1, 2, 1)
# => {:ok, [%Bimip.Message{payload: "..."}]}
```

---

### **Notes**

* `writer_device` is stored in the message record and compared to `device_id`.
* Only ETS buffer and disk/index are accessed — **Mnesia is not used**.

---

I can also make a **tiny one-line diagram** showing ETS → disk → writer_device check → result if you want.

Do you want me to do that?
