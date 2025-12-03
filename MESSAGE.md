# 📩 **Message**

The `Message` is the core **Data Transfer Object (DTO)** in the system. It secures, transports, and routes all communication. Messages may include **end-to-end encryption (E2E)** or **transport encryption only**.

It encapsulates:

* The content (`payload`)
* Security metadata (`encryption_type`, `encrypted`, `signature`)
* Address information (`from`, `to`)
* Optional server metadata (`peer`) for stream synchronization.

---

## 📘 Protobuf Definition

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
   optional int32 type = 9;         
   optional int32 transmission_mode = 10; 
   optional Peer peer = 11;
}
```

---

## 📝 Field Reference

|   Tag  | Field Name          | Type            | Description                                                                                                                                                                                                                                                                                                                                                                                                                                           | Source |
| :----: | :------------------ | :-------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----- |
|  **1** | `message_id`        | `string`        | **Client-generated ID** for message tracking and acknowledgment.                                                                                                                                                                                                                                                                                                                                                                                      | Client |
|  **2** | `from`              | `Identity`      | The **sender's** identity.                                                                                                                                                                                                                                                                                                                                                                                                                            | Client |
|  **3** | `to`                | `Identity`      | The **recipient's** identity.                                                                                                                                                                                                                                                                                                                                                                                                                         | Client |
|  **4** | `timestamp`         | `int64`         | Message creation time in **epoch milliseconds**.                                                                                                                                                                                                                                                                                                                                                                                                      | Client |
|  **5** | `payload`           | `bytes`         | Raw message content (e.g., JSON), typically used if not E2E encrypted.                                                                                                                                                                                                                                                                                                                                                                                | Client |
|  **6** | `encryption_type`   | `string`        | Security scheme: `"none"`, `"AES256"`, **`"E2E"`**.                                                                                                                                                                                                                                                                                                                                                                                                   | Client |
|  **7** | `encrypted`         | `string`        | **Base64 ciphertext** of the content, if encrypted.                                                                                                                                                                                                                                                                                                                                                                                                   | Client |
|  **8** | `signature`         | `string`        | **Base64 digital signature** for **integrity and authentication**.                                                                                                                                                                                                                                                                                                                                                                                    | Client |
|  **9** | `type`              | `int32`         | **Origin/Target Classification**: `1=SENDER`, `2=DEVICE`, `3=RECEIVER` .                                                                                                                                                                                                                                                                                                                                                                              | Client |
| **10** | `transmission_mode` | `int32`         | **Server Delivery Method**: `1=pull`, `2=push`.                                                                                                                                                                                                                                                                                                                                                                                                       | Client |
| **11** | `peer`              | `optional Peer` | **Server-assigned stream metadata** The Peer message contains server-assigned stream synchronization metadata critical for reliability. It includes the base IDs of both the sender (from) and the recipient (to), along with the server's global stream offset (offset) for the current message and the recipient's stream offset (peer_offset) as known by the server, which are continuously exchanged to maintain reliable two-way communication. | Server |

---

## 🗂️ Message Type (`type`)

| Value | Name     | Description                                                                     |
| :---: | :------- | :------------------------------------------------------------------------------ |
|   1   | SENDER   | Message originated from the sender client.                                      |
|   2   | DEVICE   | Message sent from a device (multi-device scenario).                             |
|   3   | RECEIVER | Message originated on the receiver side (e.g., a read receipt or local action). |

---

## 🚀 Transmission Mode (`transmission_mode`)

| Value | Description                                                          |
| :---: | :------------------------------------------------------------------- |
|   1   | Pull: Recipient fetches the message when online.                     |
|   2   | Push: Server pushes the message immediately to the recipient device. |

---

## 🛡️ Encryption Types

|   Type   | Description                                                |
| :------: | :--------------------------------------------------------- |
|  `none`  | No encryption, payload in clear text.                      |
| `AES256` | Transport layer encryption only.                           |
|   `E2E`  | End-to-end encryption using recipient keys (X25519 / RSA). |

---

## 📝 Example: E2E Encrypted Message (Elixir)

```elixir
payload_json = %{
  "data" => "Secret chat message",
  "emogi" => "🔒",
  "cdn_url" => "www.dcn_dart/encrypted/image.png"
} |> Jason.encode!()

encrypted_payload = MyCrypto.encrypt_for_recipient(payload_json, recipient_public_key)

request = %Bimip.Message{
  message_id: "e2e123456",
  from: %Bimip.Identity{
    eid: "a@domain.com",
    connection_resource_id: "device-abc123"
  },
  to: %Bimip.Identity{ eid: "b@domain.com" },
  timestamp: System.system_time(:millisecond),
  payload: <<>>, # empty for E2E
  encryption_type: "E2E",
  encrypted: encrypted_payload,
  signature: MyCrypto.sign(encrypted_payload, sender_private_key),
  type: 1,
  transmission_mode: 2
}
```

---


## 🌐 Multi-language Examples

**JavaScript / Node.js**

```javascript
const encryptedPayload = encryptForRecipient(JSON.stringify(payloadJson), recipientKey);

const message = new Message({
  messageId: "e2e123456",
  from: { eid: "a@domain.com", connectionResourceId: "device-abc123" },
  to: { eid: "b@domain.com" },
  timestamp: Date.now(),
  payload: Buffer.from([]),
  encryptionType: "E2E",
  encrypted: encryptedPayload,
  signature: sign(encryptedPayload, senderPrivateKey),
  type: 1,
  transmissionMode: 2
});
```

**Java**

```java
Message message = Message.newBuilder()
    .setMessageId("e2e123456")
    .setFrom(Identity.newBuilder().setEid("a@domain.com").setConnectionResourceId("device-abc123").build())
    .setTo(Identity.newBuilder().setEid("b@domain.com").build())
    .setTimestamp(System.currentTimeMillis())
    .setPayload(ByteString.EMPTY)
    .setEncryptionType("E2E")
    .setEncrypted(encryptedPayload)
    .setSignature(signature)
    .setType(1)
    .setTransmissionMode(2)
    .build();
```

**Kotlin**

```kotlin
val message = Message.newBuilder()
    .setMessageId("e2e123456")
    .setFrom(Identity.newBuilder().setEid("a@domain.com").setConnectionResourceId("device-abc123").build())
    .setTo(Identity.newBuilder().setEid("b@domain.com").build())
    .setTimestamp(System.currentTimeMillis())
    .setPayload(ByteString.EMPTY)
    .setEncryptionType("E2E")
    .setEncrypted(encryptedPayload)
    .setSignature(signature)
    .setType(1)
    .setTransmissionMode(2)
    .build()
```

**Dart**

```dart
final message = Message(
  messageId: "e2e123456",
  from: Identity(eid: "a@domain.com", connectionResourceId: "device-abc123"),
  to: Identity(eid: "b@domain.com"),
  timestamp: DateTime.now().millisecondsSinceEpoch,
  payload: Uint8List(0),
  encryptionType: "E2E",
  encrypted: encryptedPayload,
  signature: signature,
  type: 1,
  transmissionMode: 2,
);
```

**Swift**

```swift
let message = Message.with {
    $0.messageId = "e2e123456"
    $0.from = Identity.with { $0.eid = "a@domain.com"; $0.connectionResourceId = "device-abc123" }
    $0.to = Identity.with { $0.eid = "b@domain.com" }
    $0.timestamp = Int64(Date().timeIntervalSince1970 * 1000)
    $0.payload = Data()
    $0.encryptionType = "E2E"
    $0.encrypted = encryptedPayload
    $0.signature = signature
    $0.type = 1
    $0.transmissionMode = 2
}
```

