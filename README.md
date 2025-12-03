<img src="./phoenix.png" alt="Phoenix logo" />

> Peace of mind from prototype to production.

[![Build Status](https://github.com/phoenixframework/phoenix/workflows/CI/badge.svg)](https://github.com/phoenixframework/phoenix/actions/workflows/ci.yml) [![Hex.pm](https://img.shields.io/hexpm/v/phoenix.svg)](https://hex.pm/packages/phoenix) [![Documentation](https://img.shields.io/badge/documentation-gray)](https://hexdocs.pm/phoenix)


## 📚 Core Messaging Protocol Documentation

This section details the Protocol Buffer definitions for the primary communication and synchronization structures.

### 🔑 1. Identity

The `Identity` message defines a fully addressable entity within the system, allowing for targeted routing to a user's account, specific device, or hosting server node.

```protobuf
message Identity {
  string eid = 1;                              // User’s global identifier (e.g., account ID or JID)
  optional string connection_resource_id = 2;  // Identifier for a specific device or session instance
  optional string node = 3;                    // System node handling this entity (cluster context)
}
```

| Field Name | Type | Description |
| :--- | :--- | :--- |
| **`eid`** | `string` | **Entity Identifier**. The mandatory, global ID for the user or entity (e.g., account name). |
| **`connection_resource_id`** | `optional string` | **Device/Session ID**. Used to target a specific connected device or session instance belonging to the `eid`. |
| **`node`** | `optional string` | **Server Node ID**. Used for internal server routing to locate the specific cluster node managing the connection. |

-----

### 📩 2. Message

The `Message` is the core data transfer object, encapsulating content, security, and routing metadata.

```protobuf
message Message {
   string message_id = 1;
   Identity from = 2;
   Identity to = 3;
   int64 timestamp = 4;
   bytes payload = 5;
   string encryption_type = 6;
   string encrypted = 7;
   string signature = 8;
   int32 type = 9;                // 1=SENDER  2=DEVICE  3=RECEIVER
   int32 transmission_mode = 10;  // push(2)  or pull(1)
   optional Peer peer = 11;
}
```

| Field Name | Type | Description |
| :--- | :--- | :--- |
| **`message_id`** | `string` | Client-generated ID for tracking. |
| **`from`** | `Identity` | The **sender's** identity. |
| **`to`** | `Identity` | The **recipient's** identity. |
| **`timestamp`** | `int64` | Message creation time in epoch milliseconds. |
| **`payload`** | `bytes` | Raw message content (e.g., JSON), typically used if not E2E encrypted. |
| **`encryption_type`** | `string` | Security scheme: `"none"`, `"AES256"`, **`"E2E"`**. |
| **`encrypted`** | `string` | **Base64 ciphertext** of the content, if encrypted. |
| **`signature`** | `string` | **Base64 digital signature** for integrity and sender authentication. |
| **`type`** | `int32` | **Origin/Target Classification**: `1=SENDER` (chat), `2=DEVICE` (control), `3=RECEIVER` (notification). |
| **`transmission_mode`** | `int32` | **Server Delivery Method**: `1=pull`, `2=push`. |
| **`peer`** | `optional Peer` | **Server-assigned stream metadata** (offsets) for reliability. |

-----

### 🤝 3. Peer

The `Peer` message carries synchronization information vital for stream management and reliability, typically populated by the server.

```protobuf
message Peer {
  string from = 1;
  string to = 2;
  int64 offset = 3;
  int64 peer_offset = 4;
}
```

| Field Name | Type | Description |
| :--- | :--- | :--- |
| **`from`** | `string` | The **base ID** (`eid`) of the message sender in the context of the stream. |
| **`to`** | `string` | The **base ID** (`eid`) of the message recipient in the context of the stream. |
| **`offset`** | `int64` | The **server-assigned global stream ID** of the current message or synchronization point. |
| **`peer_offset`** | `int64` | The **last acknowledged offset** of the opposing peer's stream, crucial for flow control and reliability. |

-----

### ✅ 4. MessagePeerAckSignal

The `MessagePeerAckSignal` is the acknowledgment mechanism used by the system. It is sent back to the original message sender to confirm delivery, update status, and convey critical, server-generated stream synchronization data.

```protobuf
message MessagePeerAckSignal {
  Identity to = 1;
  Identity from = 2;
  string message_id = 3;
  int32 method = 4;
  int64 timestamp = 5;
  int32 status_code = 6;
  Peer peer = 7;
}
```

| Field Name | Type | Description |
| :--- | :--- | :--- |
| **`to`** | `Identity` | The **original message sender** (recipient of the ACK). |
| **`from`** | `Identity` | The **original message recipient** (sender of the ACK). |
| **`message_id`** | `string` | The **ID of the message** being acknowledged. |
| **`method`** | `int32` | The method or action related to the ACK (e.g., delivered, read, requested offset). |
| **`timestamp`** | `int64` | Time the acknowledgment was generated (server time recommended). |
| **`status_code`** | `int32` | **HTTP-like status code** (e.g., 200 for OK, 400 for Failure). |
| **`peer`** | `Peer` | **CRITICAL:** Contains the server-generated stream offsets (`offset` and `peer_offset`) required for the sender to synchronize its local stream state. |

-----

Would you like to review this documentation or perhaps define the status codes you plan to use for the `MessagePeerAckSignal`?

### Copyright and License

Copyright (c) 2014, Chris McCord.

Phoenix source code is licensed under the [MIT License](LICENSE.md).
