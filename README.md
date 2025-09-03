## BIMip-Foundation Messaging Protocol Specification

**Status:** Draft
**Category:** Standards Track
**Author:** Paul Aigokhai Olukayode
**Created:** 2025-08-30

-----

### Table of Contents

1.  Introduction
2.  Terminology
3.  Protocol Overview
4.  Message Types
    4.1. Awareness Messages
    4.2. PingPong Messages
    4.3. Token Revoke Messages
    4.4. Subscriber Messages
    4.5. Block Subscriber Messages
    4.6. Logout Messages
    4.7. Error Messages
    4.8. Chat Messaging
5.  Protocol Buffers Definitions
    5.1. Identity
    5.2. Awareness
    5.3. PingPong
    5.4. Token Revoke
    5.5. Subscriber
    5.6. Block Subscriber
    5.7. Logout
    5.8. Error
    5.9. Chat Messaging
    5.10. Version Updates
    5.11. MessageScheme Envelope
6.  Semantics
7.  Status & Type Tables
8.  Example Exchanges
    8.1. Awareness Example
    8.2. PingPong Example
    8.3. Token Revoke Example
    8.4. Chat Messaging Examples
9.  Security Considerations
10. IANA Considerations
11. References

-----

### 1\. Introduction

This document specifies the messaging protocol for communication between clients and the server, defining how text messages, media messages, acknowledgments, and version updates are structured and exchanged.

The **Awareness Protocol (AWP)** defines a lightweight message-based system for communicating user and device presence ("awareness") between entities. The **PingPong Protocol (PPG)** provides a mechanism to verify connectivity between devices and servers, measure latency, and detect lost connections. The **Token Revoke Protocol (TRP)** provides JWT-based session revocation and logout management. **Subscriber** and **Block Subscriber** protocols manage user access and communication control. The **Chat Messaging Protocol (CMP)** provides 1:1 user-to-user message exchange, supporting text and media payloads, with acknowledgment tracking for delivery and read status. All protocols are part of **BIMip (Binary Interface for Messaging & Internet Protocol)**, optimized for internal service and mobile-backend communication.

-----

### 2\. Terminology

  * **Epohai Identifier (EID):** Unique user identifier (e.g., `alice@domain.com`).
  * **Device EID:** Identifier for a specific device under a user EID.
  * **Requester:** Entity asking about awareness or connectivity.
  * **Responder:** Entity providing awareness or ping response.
  * **Notification:** Proactive awareness update sent without a request.
  * **Route:** Logical identifier in the `MessageScheme` envelope indicating the payload schema.

-----

### 3\. Protocol Overview

Primary message categories:

  * **Awareness Messages:** Query or notify user/device awareness state.
  * **PingPong Messages:** Verify device-to-server connectivity.
  * **Token Revoke Messages:** Logout/revoke device sessions.
  * **Subscriber Messages:** Add/remove subscribers.
  * **Block Subscriber Messages:** Block unwanted communication.
  * **Logout Messages:** Disconnect a device/session.
  * **Error Messages:** Standardized error reporting.
  * **Chat Messaging:** 1:1 text and media exchange with acknowledgment.

All messages use **Protocol Buffers** and are wrapped in a `MessageScheme` envelope for routing.

-----

### 4\. Message Types

#### 4.1 Awareness Messages

  * **AwarenessRequest** – Query another user/device for their current presence/status.
  * **AwarenessResponse** – Reply to `AwarenessRequest` with status, location, and intention.
  * **AwarenessNotification** – Proactively notify other users/devices of awareness state.

#### 4.2 PingPong Messages

  * **PingPong REQUEST** – Device → Server connectivity check.
  * **PingPong RESPONSE** – Server → Device response with latency/status.

#### 4.3 Token Revoke Messages

  * **TokenRevokeRequest** – Initiates logout for specific devices or all devices of a user.
  * **TokenRevokeResponse** – Confirms whether revocation succeeded.

#### 4.4 Subscriber Messages

  * **SubscriberAddRequest** – Request to add a subscriber under a user.
  * **SubscriberAddResponse** – Confirms addition with status and resource ID.

#### 4.5 Block Subscriber Messages

  * **BlockSubscriber REQUEST** – Request to block a subscriber.
  * **BlockSubscriber RESPONSE** – Confirms status of the block.

#### 4.6 Logout Messages

  * **Logout REQUEST** – Client/device requests logout/disconnect.
  * **Logout RESPONSE** – Server confirms logout status.

#### 4.7 Error Messages

  * **ErrorMessage** – Protocol-level errors (invalid requests, schema mismatches, or authorization failures).

#### 4.8 Chat Messaging

  * **TextPayloadRequest** – A text message request with metadata.
  * **MediaPayloadRequest** – A media message request with URLs and metadata.
  * **ChatMessage** – The canonical message entity sent to receivers and stored as a copy by the sender.
  * **AcknowledgmentRequest/Response** – Used by sender/receiver to confirm message status.
  * **VersionUpdate** - Used when the server assigns or updates version numbers.

-----

### 5\. Protocol Buffers Definitions

#### 5.1 Identity

```proto
message Identity {
  string eid                     = 1;  // user or device identifier (e.g. "a@domain.com")
  string connection_resource_id  = 2;  // optional client session or connection ID
}
```

#### 5.2 Awareness

```proto
message AwarenessRequest {
  Identity from = 1;
  Identity to = 2;
  int64 awareness_identifier = 3;
  int64 timestamp = 4;
}

message AwarenessResponse {
  Identity from = 1;
  Identity to = 2;
  string awareness_identifier = 3;
  int32 status = 4;
  double latitude = 5;
  double longitude = 6;
  int32 awareness_intention = 7;
  int64 timestamp = 8;
}

message AwarenessNotification {
  Identity from = 1;
  Identity to = 2;
  int32 status = 3;
  int64 last_seen = 4;
  double latitude = 5;
  double longitude = 6;
  int32 awareness_intention = 7;
}
```

#### 5.3 PingPong

```proto
message PingPong {
  Identity to = 1;
  int32 type = 2;      // 1=REQUEST, 2=RESPONSE
  int32 status = 3;    // 1=PENDING ... 6=UNREACHABLE
  int64 request_time = 4;
  int64 response_time = 5;
  int64 rtt = 6;
}
```

#### 5.4 Token Revoke

```proto
message TokenRevokeRequest {
  Identity to = 1;
  string token = 2;
  int32 type = 3;        // 1=REQUEST, 2=RESPONSE
  int64 timestamp = 4;
  int64 rtt = 5;
}

message TokenRevokeResponse {
  Identity to = 1;
  int32 status = 2;      // 1=SUCCESS, 2=FAILED
  int32 type = 3;        // 1=REQUEST, 2=RESPONSE
  int64 timestamp = 4;
  int64 rtt = 5;
}
```

#### 5.5 Subscriber

```proto
message SubscriberAddRequest {
  Identity owner = 1;
  Identity subscriber = 2;
  string nickname = 3;
  string group = 4;
  string subscriber_resource_id = 5;
  int64 timestamp = 6;
}

message SubscriberAddResponse {
  Identity owner = 1;
  Identity subscriber = 2;
  string subscriber_resource_id = 3;
  int32 status = 4;
  string message = 5;
  int64 timestamp = 6;
}
```

#### 5.6 Block Subscriber

```proto
message BlockSubscriber {
  Identity owner = 1;
  Identity subscriber = 2;
  int32 type = 3;    // 1=REQUEST, 2=RESPONSE
  int32 status = 4;
  string message = 5;
  int64 timestamp = 6;
}
```

#### 5.7 Logout

```proto
message Logout {
  Identity entity = 1;
  int32 type = 2;
  int32 status = 3;
  int64 timestamp = 4;
}
```

#### 5.8 Error

```proto
message ErrorMessage {
  int32 code = 1;
  string message = 2;
  string route = 3;
  string details = 4;
}
```

#### 5.9 Chat Messaging

##### 5.9.1 Payload Requests

Payload requests are client-initiated messages sent to either the messaging server or the CDN server.

// **TextPayloadRequest**

```proto
message TextPayloadRequest {
  Identity from           = 1;
  repeated Identity to    = 2;
  string content          = 3;
  string message_id_local = 4;
  string text_size_count  = 5;
  int64 created_at        = 6;
}
```

Sent by the client when submitting a text message.

// **MediaPayloadRequest**

```proto
message MediaPayloadRequest {
  Identity from           = 1;
  repeated Identity to    = 2;
  string message_id_local = 3;
  string chat_id          = 4;
  int32  type             = 5;  // 1=image | 2=audio | 3=video | 4=file
  string caption          = 6;
  string thumbnail_url    = 7;
  string cdn_url_id       = 8;
  int64 media_size_bytes  = 9;
  int64 created_at        = 10;
}
```

Sent to the CDN server for handling media uploads and metadata.

##### 5.9.2 Canonical ChatMessage

This message represents a chat message that is sent to the receiver(s) and also kept as a copy by the sender for local storage and synchronization.

```proto
message ChatMessage {
  string message_id       = 1;  // server-assigned global ID
  string message_id_local = 2;  // client temporary ID
  int64  version          = 3;  // server-assigned chat sequence
  string chat_id          = 4;  // conversation ID
  Identity from           = 5;  // message sender
  repeated Identity to    = 6;  // message receivers
  string author           = 7;  // "sender" or "receiver"

  oneof payload {
    string text       = 8;
    string media_url  = 9;  // CDN links
  }

  // Acknowledgment status codes:
  // Sender view:    4=sent (initial), 2=delivered, 3=read
  // Receiver view: 1=unread (initial), 2=delivered, 3=read
  int32 acknowledgment            = 10;
  int64 acknowledgment_timestamp  = 11;

  int64 created_at                = 12; // client-side creation
  int64 server_received_at        = 13; // server timestamp
}
```

**Sender copy:** Keeps message with `author = "sender"`, `acknowledgment = 4` (sent).
**Receiver copy:** Receives message with `author = "receiver"`, `acknowledgment = 1` (unread).

##### 5.9.3 Acknowledgment Flow

Acknowledgments are exchanged to track delivery and read states between sender and receiver.

// **AcknowledgmentRequest**

```proto
message AcknowledgmentRequest {
  Identity from     = 1;  // sending client or receiving client depending on flow
  Identity to       = 2;  // original sender or receiver
  string message_id = 3;

  // Status codes:
  // Sender side: 1=unread | 2=delivered | 3=read
  // Receiver side: 4=sent (initial) | 2=delivered | 3=read
  int32  status     = 4;
  int64  timestamp  = 5;  // time of acknowledgment
}
```

// **AcknowledgmentResponse**

```proto
message AcknowledgmentResponse {
  Identity from     = 1;  // sender/receiver depending on flow
  Identity to       = 2;  // receiver/sender depending on flow
  string message_id = 3;

  // Status codes:
  // Sender side: 1=unread | 2=delivered | 3=read
  // Receiver side: 4=sent (initial) | 2=delivered | 3=read
  int32  status     = 4;
  int64  timestamp  = 5;  // time of acknowledgment
}
```

#### 5.10 Version Updates

Used when the server assigns or updates version numbers.

// **VersionUpdate**

```proto
message VersionUpdate {
  Identity to           = 1;  // original sender or receiver
  string message_id     = 2;
  string message_id_local = 3;  // client temporary ID
  int32 type            = 4;   // 1 = REQUEST, 2 = RESPONSE
  int64 timestamp       = 5;
  int32 status          = 6;   // 1 = sent, 2 = updated
}
```

**Flow:**

1.  Server sends `VersionUpdate` (REQUEST) to sender with assigned version.
2.  Sender updates local copy and replies with `VersionUpdate` (RESPONSE).
3.  Server then pushes `ChatMessage` with version to receivers.

#### 5.11 MessageScheme Envelope

All payloads are wrapped in the `MessageScheme` envelope for routing.

// **MessageScheme**

```proto
message MessageScheme {
  int64 route = 1;

  oneof payload {
    AwarenessNotification awareness_notification = 2;
    AwarenessResponse awareness_response = 3;
    AwarenessRequest awareness_request = 4;
    ErrorMessage error_message = 5;
    PingPong pingpong_message = 6;
    TokenRevokeRequest token_revoke_request = 7;
    TokenRevokeResponse token_revoke_response = 8;
    SubscriberAddRequest subscriber_add_request = 9;
    SubscriberAddResponse subscriber_add_response = 10;
    BlockSubscriber block_subscriber = 11;
    Logout logout = 12;
    ChatMessage chat_message = 13;
    TextPayloadRequest text_payload_request = 14;
    MediaPayloadRequest media_payload_request = 15;
    AcknowledgmentRequest acknowledgment_request = 16;
    AcknowledgmentResponse acknowledgment_response = 17;
  }
}
```

-----

### 6\. Semantics

  * All requests must include a valid `Identity` for both `from` and `to`.
  * `timestamp` fields are Unix time in milliseconds.
  * `status` codes vary per message type and must follow defined tables.
  * `MessageScheme` route identifiers are used to multiplex payloads.
  * Chat messages are **1:1 only**; group support is out of scope.

-----

### 7\. Status & Type Tables

**PingPong Status Codes**

| Status | Meaning |
| ------ | ----------- |
| 1 | PENDING |
| 2 | OK |
| 3 | FAIL |
| 4 | TIMEOUT |
| 5 | ERROR |
| 6 | UNREACHABLE |

**PingPong Type Codes**

| Type | Meaning |
| ---- | -------- |
| 1 | REQUEST |
| 2 | RESPONSE |

**TokenRevoke Type Codes**

| Type | Meaning |
| ---- | -------- |
| 1 | REQUEST |
| 2 | RESPONSE |

**Logout Status Codes**

| Status | Meaning |
| ------ | ---------------- |
| 1 | DISCONNECT – Client disconnect |
| 2 | FAIL – Logout failed |
| 3 | SUCCESS – Logout succeeded |

**Chat Acknowledgment Codes**

| Status | Meaning |
| ------ | ----------------------------- |
| 1 | UNREAD (sender-side) |
| 2 | DELIVERED (both sides) |
| 3 | READ (both sides) |
| 4 | SENT (receiver initial state) |

-----

### 8\. Example Exchanges

#### 8.1 Awareness Example

```elixir
req = %Dartmessaging.AwarenessRequest{
  from: %Dartmessaging.Identity{eid: "alice@domain.com"},
  to: %Dartmessaging.Identity{eid: "bob@domain.com"},
  awareness_identifier: 12345,
  timestamp: System.system_time(:millisecond)
}
```

#### 8.2 PingPong Example

```elixir
request_time = System.system_time(:millisecond)
ping = %Dartmessaging.PingPong{
  to: %Dartmessaging.Identity{eid: "client@domain.com"},
  type: 1,
  status: 1,
  request_time: request_time
}

response_time = System.system_time(:millisecond)
ping_response = %Dartmessaging.PingPong{
  to: ping.to,
  type: 2,
  status: 2,
  request_time: ping.request_time,
  response_time: response_time,
  rtt: response_time - ping.request_time
}
```

#### 8.3 Token Revoke Example

```elixir
request_time = System.system_time(:millisecond)
revoke_request = %Dartmessaging.TokenRevokeRequest{
  to: %Dartmessaging.Identity{eid: "client@domain.com"},
  token: "jwt-token-string",
  type: 1,
  timestamp: request_time
}

response_time = System.system_time(:millisecond)
revoke_response = %Dartmessaging.TokenRevokeResponse{
  to: revoke_request.to,
  status: 1,
  type: 2,
  timestamp: response_time,
  rtt: response_time - request_time
}
```

#### 8.4 Chat Messaging Examples

**Example 1: Sending a Text Message**

  * Sender → Server

<!-- end list -->

```elixir
# Sender creates a text message
msg = %Dartmessaging.TextPayloadRequest{
  from: %Dartmessaging.Identity{eid: "alice@domain.com"},
  to: [%Dartmessaging.Identity{eid: "bob@domain.com"}],
  content: "Hey Bob!",
  message_id_local: "local-123",
  text_size_count: "8",
  created_at: System.system_time(:millisecond)
}
```

**Example 2: Server Sends ChatMessage to Receiver**

  * Server wraps into ChatMessage

<!-- end list -->

```elixir
chat = %Dartmessaging.ChatMessage{
  message_id: "srv-456",
  message_id_local: msg.message_id_local,
  version: 1,
  chat_id: "chat-abc",
  from: msg.from,
  to: msg.to,
  author: "sender",
  text: msg.content,
  acknowledgment: 4, # SENT
  created_at: msg.created_at,
  server_received_at: System.system_time(:millisecond)
}
```

**Example 3: Receiver Sends Acknowledgment**

  * Receiver acknowledges delivery

<!-- end list -->

```elixir
ack = %Dartmessaging.AcknowledgmentRequest{
  from: %Dartmessaging.Identity{eid: "bob@domain.com"},
  to: msg.from,
  message_id: chat.message_id,
  status: 2, # DELIVERED
  timestamp: System.system_time(:millisecond)
}
```

-----

### 9\. Security Considerations

  * Authenticate awareness requests to prevent spoofing.
  * Share sensitive metadata (e.g., location) only with authorized parties.
  * Rate-limit notifications to prevent flooding.
  * Use TLS for all transport layers.
  * Chat messages must be encrypted end-to-end where applicable.

-----

### 10\. IANA Considerations

  * Introduces new namespaces: awareness, pingpong, subscriber, logout, chat.
  * No IANA registry actions required currently.

-----

### 11\. References

  * [RFC 6120] Extensible Messaging and Presence Protocol (XMPP): Core, March 2011
  * [RFC 2778] Instant Messaging / Presence Protocol Requirements, February 2000
