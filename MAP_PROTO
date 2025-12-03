# **Bimip Protobuf**

## **Purpose**

Defines messaging payloads for the BIMip system. Supports multi-language usage (Elixir, JavaScript, Dart, Swift, Java/Kotlin). This doc provides **field mapping, naming conventions, and examples**.

---

## **1. Message Field Mapping**

| **Proto Field**     | **Elixir**             | **JavaScript (protobufjs)**                 | **Java / Kotlin**                      | **Dart**                  | **Swift**                       |
| ------------------- | ---------------------- | ------------------------------------------- | -------------------------------------- | ------------------------- | ------------------------------- |
| `message_id`        | `message_id`           | `messageId`                                 | `setMessageId(...)`                    | `messageId`               | `messageID`                     |
| `from`              | `from: %Identity{...}` | `from: Identity.create({...})`              | `setFrom(...)`                         | `from: Identity(...)`     | `$0.from = Identity.with {...}` |
| `to`                | `to: %Identity{...}`   | `to: Identity.create({...})`                | `setTo(...)`                           | `to: Identity(...)`       | `$0.to = Identity.with {...}`   |
| `timestamp`         | `timestamp`            | `timestamp`                                 | `setTimestamp(...)`                    | `timestamp`               | `$0.timestamp = ...`            |
| `payload`           | `payload`              | `payload: Buffer.from(JSON.stringify(...))` | `setPayload(ByteString.copyFrom(...))` | `payload: Uint8List(...)` | `$0.payload = Data(...)`        |
| `encryption_type`   | `encryption_type`      | `encryptionType`                            | `setEncryptionType(...)`               | `encryptionType`          | `encryptionType`                |
| `encrypted`         | `encrypted`            | `encrypted`                                 | `setEncrypted(...)`                    | `encrypted`               | `encrypted`                     |
| `signature`         | `signature`            | `signature`                                 | `setSignature(...)`                    | `signature`               | `signature`                     |
| `type`              | `type`                 | `type`                                      | `setType(...)`                         | `type`                    | `type`                          |
| `transmission_mode` | `transmission_mode`    | `transmissionMode`                          | `setTransmissionMode(...)`             | `transmissionMode`        | `transmissionMode`              |
| `peer`              | `peer`                 | `peer: Peer.create(...)`                    | `setPeer(...)`                         | `peer`                    | `peer`                          |

---

## **2. Identity Message Mapping**

| **Proto Field**          | **Elixir**               | **JavaScript**         | **Java / Kotlin**              | **Dart**               | **Swift**              |
| ------------------------ | ------------------------ | ---------------------- | ------------------------------ | ---------------------- | ---------------------- |
| `eid`                    | `eid`                    | `eid`                  | `setEid(...)`                  | `eid`                  | `eid`                  |
| `connection_resource_id` | `connection_resource_id` | `connectionResourceId` | `setConnectionResourceId(...)` | `connectionResourceId` | `connectionResourceId` |
| `node`                   | `node`                   | `node`                 | `setNode(...)`                 | `node`                 | `node`                 |

---

## **3. Example Usage**

### **Elixir**

```elixir
request = %Bimip.Message{
  message_id: "MSG123",
  from: %Bimip.Identity{eid: "a@domain.com", connection_resource_id: "device-1"},
  to: %Bimip.Identity{eid: "b@domain.com"},
  timestamp: System.system_time(:millisecond),
  payload: Jason.encode!(%{data: "Hello!"}),
  encryption_type: "E2E",
  encrypted: "",
  signature: "",
  type: 1,
  transmission_mode: 2
}

message = %Bimip.MessageScheme{
  route: 6,
  payload: {:message, request}
}

binary = Bimip.MessageScheme.encode(message)
hex = Base.encode16(binary, case: :upper)
```

### **JavaScript**

```js
const Identity = root.lookupType("bimip.Identity");
const Message = root.lookupType("bimip.Message");
const MessageScheme = root.lookupType("bimip.MessageScheme");

const from = Identity.create({ eid: "a@domain.com", connection_resource_id: "device-1" });
const to = Identity.create({ eid: "b@domain.com" });

const request = Message.create({
  messageId: "MSG123",
  from,
  to,
  timestamp: Date.now(),
  payload: Buffer.from(JSON.stringify({ data: "Hello!" })),
  encryptionType: "E2E",
  encrypted: "",
  signature: "",
  type: 1,
  transmissionMode: 2
});

const messageScheme = MessageScheme.create({
  route: 6,
  message: request
});

const binary = MessageScheme.encode(messageScheme).finish();
const hex = Buffer.from(binary).toString("hex").toUpperCase();
```

---

## **4. Naming Rules**

1. **Proto3 fields**: snake_case by default.
2. **Elixir**: use snake_case as-is.
3. **JavaScript (protobufjs)**: convert snake_case → camelCase (`message_id` → `messageId`).
4. **Dart**: follows camelCase (`messageId`).
5. **Swift**: fields like `message_id` → `messageID`.
6. **Java/Kotlin**: access via getters/setters (`setMessageId(...)`).
