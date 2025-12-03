
## 🤝 Peer

The `Peer` message carries synchronization information between two entities (peers). It primarily holds **stream offsets** vital for reliability, flow control, and preventing message duplication/loss.

### Protobuf Definition

```protobuf
message Peer {
  string from = 1;
  string to = 2;
  int64 offset = 3;
  int64 peer_offset = 4;
}
```

### Field Reference

| Tag | Field Name | Type | Description | Primary Role |
| :---: | :--- | :--- | :--- | :--- |
| **1** | `from` | `string` | The **base ID** (`eid`) of the message sender in the context of the stream. | Context |
| **2** | `to` | `string` | The **base ID** (`eid`) of the message recipient in the context of the stream. | Context |
| **3** | `offset` | `int64` | The **global stream ID** assigned by the server to the message this `Peer` object is associated with. | **Message Position** |
| **4** | `peer_offset` | `int64` | The **highest global stream offset that the receiving peer has successfully processed/acknowledged**. | **Recipient Status** |

