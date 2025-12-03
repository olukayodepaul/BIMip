## 📩 Message

`Message` as the core Data Transfer Object (DTO), the Message secures and routes all communication. It encapsulates the content, necessary security metadata (encryption and signature), and address information such as the sender (from) and recipient (to).

### Protobuf Definition

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
   int32 type = 9;         
   int32 transmission_mode = 10; 
   optional Peer peer = 11;
}
```

### Field Reference

| Tag | Field Name | Type | Description | Source |
| :---: | :--- | :--- | :--- | :--- |
| **1** | `message_id` | `string` | **Client-generated ID** for message tracking and acknowledgment. | Client |
| **2** | `from` | `Identity` | The **sender's** identity. | Client |
| **3** | `to` | `Identity` | The **recipient's** identity. | Client |
| **4** | `timestamp` | `int64` | Message creation time in **epoch milliseconds**. | Client |
| **5** | `payload` | `bytes` | Raw message content (e.g., JSON), typically used if not E2E encrypted. | Client |
| **6** | `encryption_type` | `string` | Security scheme: `"none"`, `"AES256"`, **`"E2E"`**. | Client |
| **7** | `encrypted` | `string` | **Base64 ciphertext** of the content, if encrypted. | Client |
| **8** | `signature` | `string` | **Base64 digital signature** for **integrity and authentication**. | Client |
| **9** | `type` | `int32` | **Origin/Target Classification**: `1=SENDER`, `2=DEVICE`, `3=RECEIVER` . | Client |
| **10** | `transmission_mode` | `int32` | **Server Delivery Method**: `1=pull`, `2=push`. | Client |
| **11** | `peer` | `optional Peer` | **Server-assigned stream metadata** containing stream offsets for reliability. | Server |
