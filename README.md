# BIMip-Foundation (RFC-DRAFT) 

**Status:** Draft  
**Category:** Standards Track  
**Author:** Paul Aigokhai Olukayode  
**Created:** 2025-08-30

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Terminology](#2-terminology)
3. [Protocol Overview](#3-protocol-overview)
4. [Message Types](#4-message-types)
   - [4.1 Awareness Messages](#41-awareness-messages)
   - [4.2 PingPong Messages](#42-pingpong-messages)
   - [4.3 Token Revoke Messages](#43-token-revoke-messages)
5. [Protocol Buffers Definitions](#5-protocol-buffers-definitions)
   - [5.1 Awareness](#51-awareness)
   - [5.2 PingPong](#52-pingpong)
   - [5.3 TokenRevoke](#53-tokenrevoke)
6. [Semantics](#6-semantics)
   - [6.1 Awareness](#61-awareness)
   - [6.2 PingPong](#62-pingpong)
   - [6.3 TokenRevoke](#63-tokenrevoke)
7. [Example Exchanges](#7-example-exchanges)
   - [7.1 Awareness](#71-awareness)
   - [7.2 PingPong](#72-pingpong)
   - [7.3 TokenRevoke](#73-tokenrevoke)
8. [Security Considerations](#8-security-considerations)
9. [IANA Considerations](#9-iana-considerations)
10. [References](#10-references)

---

## 1. Introduction

The Awareness Protocol (AWP) defines a lightweight message-based system for communicating user and device presence ("awareness") between entities.  
It allows one entity to query the awareness of another, receive responses, and subscribe to notifications about awareness changes.

The PingPong Protocol (PPG) provides a standardized mechanism to verify connectivity between two entities, measure latency, and detect lost connections.

The Token Revoke Protocol (TRP) defines a mechanism for logging out specific devices or all devices of a user using JWT-based authentication.

Together, they form part of **BIMip (Binary Interface for Messaging & Internet Protocol).**

---

## 2. Terminology

- **Epohai Identifier (EID):** A unique identifier for a user, e.g., `alice@domain.com`
- **Device EID:** Identifier for a specific device under a user EID
- **Requester:** The entity asking about awareness or connectivity
- **Responder:** The entity providing awareness or ping response
- **Notification:** A proactive awareness update sent without a request
- **Route:** Logical identifier in the wrapper indicating which payload schema is carried

---

## 3. Protocol Overview

The protocol defines three categories of primary message types:

### Awareness Messages

- **AwarenessRequest** – Sent by a requester to query another entity’s awareness state
- **AwarenessResponse** – Sent by a responder to return the requested awareness state
- **AwarenessNotification** – Sent proactively to notify subscribers about awareness changes

### PingPong Messages

- **PingPong (REQUEST)** – Sent to check connectivity and measure round-trip latency
- **PingPong (RESPONSE)** – Sent as a reply to indicate success or failure

### Token Revoke Messages

- **TokenRevoke (REQUEST)** – Sent by client or server to revoke a device or session
- **TokenRevoke (RESPONSE)** – Sent to confirm revocation

Messages are encoded using **[Protocol Buffers](https://protobuf.dev/)** for compact and interoperable serialization.  
All messages are wrapped in a `MessageScheme` **envelope** that contains a `route` and a `oneof payload`. The `route` allows the client or server to know which schema to decode without guessing.

---

## 4. Message Types

### 4.1 Awareness Messages

- **AwarenessRequest** – Query awareness state
- **AwarenessResponse** – Return awareness state
- **AwarenessNotification** – Proactive awareness updates

### 4.2 PingPong Messages

- **PingPong (REQUEST)** – Sent to verify connectivity and measure round-trip time
- **PingPong (RESPONSE)** – Sent as a reply to indicate success/failure and timing

### 4.3 Token Revoke Messages

- **TokenRevoke (REQUEST)** – Initiates logout for a device or all devices under a user EID
- **TokenRevoke (RESPONSE)** – Confirms revocation

---

## 5. Protocol Buffers Definitions

```proto
syntax = "proto3";
package dartmessaging;

message Identity {
  string eid = 1;                     // Unique user identifier
  string connection_resource_id = 2;  // Unique ID for this specific device/session
}

```

### 5.1 Awareness

```proto

message Identity {
  string eid = 1; // User/entity identifier
}

// AwarenessRequest: Query the awareness of another entity
message AwarenessRequest {
  Identity from = 1;                 // Requesting entity (EID)
  Identity to = 2;                   // Target entity (EID)
  int64 awareness_identifier = 3;  // Unique request identifier (System.system_time(:millisecond))
  int64 timestamp = 4;             // Unix UTC timestamp (ms) of request
}

message AwarenessResponse {
  Identity from = 1;                    // Responding entity (EID)
  Identity to = 2;                      // Original requester (EID)
  string awareness_identifier = 3;    // Must match AwarenessRequest.awareness_identifier
  int32 status = 4;                   // Awareness state: 1=ONLINE, 2=OFFLINE, 3=AWAY, 4=DND, 5=BUSY, 6=INVISIBLE, 7=NOT_FOUND, 8=UNKNOWN
  double latitude = 5;                // Optional
  double longitude = 6;               // Optional
  int32 awareness_intention = 7;      // Optional, defaults to 0
  int64 timestamp = 8;                // Unix UTC timestamp (ms) of response
}

message AwarenessNotification {
  Identity from = 1;                     // Entity whose awareness changed
  Identity to = 2;                       // Target entity (EID)
  int32 status = 3;                     // Awareness state: 1=ONLINE, 2=OFFLINE, 3=AWAY, 4=DND, 5=BUSY, 6=INVISIBLE, 7=NOT_FOUND, 8=UNKNOWN
  int64 last_seen = 4;                  // Unix UTC timestamp
  double latitude = 5;                  // Optional
  double longitude = 6;                 // Optional
  int32 awareness_intention = 7;        // Optional, defaults to 0
}



```
### 5.2 PingPong

```proto

message Identity {
  string eid = 1; // User/entity identifier
}


message PingPong {
  Identity from = 1;          // Sender entity (EID)
  Identity to = 2;            // Recipient entity (EID)
  int64 type = 3;           // 1 = REQUEST, 2 = RESPONSE
  int64 status = 4;         // 1 = PENDING, 2 = SUCCESS, 3 = FAIL, 4 = TIMEOUT, 5 = BLOCKED, 6 = UNREACHABLE
  int64 request_time = 5;   // Unix UTC timestamp of request (ms)
  int64 response_time = 6;  // Unix UTC timestamp of response (ms)
}
````

### 5.3 TokenRevoke

```proto

message Identity {
  string eid = 1;                     // Unique user identifier
  string connection_resource_id = 2;  // Unique ID for this specific device/session
}


message TokenRevokeRequest {
  Identity to = 1;       
  string token = 2;       // raw JWT string
  int64 timestamp = 3;    // for auditing/logging
}

message TokenRevokeResponse {
  Identity to = 1;        
  int32 status = 2;       // 1 = SUCCESS, 2 = FAILED
  int64 timestamp = 3;
}

```

### 5.4  Error

```proto
// Standardized error message
message ErrorMessage {
  int32 code = 1;
  string message = 2;
  string route = 3;
  string details = 4;
}

```

### 5.5 Add Contact

```proto

// Identity structure for user
message Identity {
  string eid = 1;          // User identifier
}

// Contact request
message ContactAddRequest {
  Identity owner = 1;            // The user who owns the contact
  Identity subscriber = 2;       // The user being added as a contact
  string nickname = 3;           // Optional nickname
  string group = 4;              // Optional group/folder
  string contact_resource_id = 5; // Unique ID to bind request and response
  int64 timestamp = 6;           // For auditing/logging
}

// Contact response
message ContactAddResponse {
  Identity owner = 1;             // Owner of the contact
  Identity subscriber = 2;        // The contact being added
  string contact_resource_id = 3; // Echoed from request
  int32 status = 4;               // 1 = SUCCESS, 2 = FAILED
  string message = 5;             // Optional error or confirmation message
  int64 timestamp = 6;            // For auditing/logging
}

```

### Block Subscriber 5.6

```proto
// Identity structure for user/entity
message Identity {
  string eid = 1; // User/entity identifier
}

// Block subscriber action
message BlockSubscriber {
  Identity owner = 1;       // The user performing the block
  Identity subscriber = 2;  // The user being blocked
  int32 type = 3;           // 1 = REQUEST, 2 = RESPONSE
  int32 status = 4;         // 1 = SUCCESS, 2 = FAILED (used only if type = RESPONSE)
  string message = 5;       // Optional message (used only if type = RESPONSE)
  int64 timestamp = 6;      // Unix UTC timestamp (ms) of action
}

```

### MessageScheme Envelope

```proto
// Route numbers for MessageScheme:
// 1 -> Logical route identifier
// 2 -> AwarenessNotification
// 3 -> AwarenessResponse
// 4 -> AwarenessRequest
// 5 -> ErrorMessage
// 6 -> PingPong
// 7 -> TokenRevokeRequest
// 8 -> TokenRevokeResponse
// 9 -> ContactAddRequest
// 10 -> ContactAddResponse
// 11 -> BlockSubscriber 

// MessageScheme: Envelope for routing multiple schemas
message MessageScheme {
  int64 route = 1;  // Logical route identifier

  oneof payload {
    AwarenessNotification awareness_notification = 2;
    AwarenessResponse awareness_response = 3;
    AwarenessRequest awareness_request = 4;
    ErrorMessage error_message = 5;
    PingPong pingpong_message = 6;
    TokenRevokeRequest token_revoke_request = 7;
    TokenRevokeResponse token_revoke_response = 8;
    ContactAddRequest contact_add_request = 9;
    ContactAddResponse contact_add_response = 10;
    BlockSubscriber block_subscriber = 11;
  }
}
```

---

## 6. Semantics

### 6.1 Awareness

- Requests **MUST** be answered with responses unless blocked/unauthorized.
- Responses **MUST** echo `request_id`.
- Notifications **MAY** be sent proactively without acknowledgment.

### 6.2 PingPong

- A PingPong **REQUEST** is used to test connection health.
- A PingPong **RESPONSE** **MUST** be returned with same timestamps.
- `status` indicates if the connectivity check was successful or failed.

### 6.3 TokenRevoke

- **from:** identifies the initiator of the revoke. Can include `device_eid` if action originates from a specific device.
- **to:** identifies the target user or device. Including `device_eid` targets a specific device; omitting it applies to all devices under the user EID.
- **type:** distinguishes between a **REQUEST** (initiated by client/server) and a **RESPONSE** (confirmation by server).
- **timestamp:** marks when the revoke request or confirmation occurred.
- Servers **MUST** immediately invalidate any revoked sessions and optionally notify other devices.

---

## 7. Example Exchanges

### 7.1 Awareness Example

```elixir
def handle_cast({:fan_out_to_children, {owner_device_id, eid, awareness}}, state) do
  notification = %Dartmessaging.AwarenessNotification{
    from: "#{awareness.owner_eid}",
    last_seen: DateTime.to_unix(awareness.last_seen, :second),
    status: awareness.status,
    latitude: awareness.latitude,
    longitude: awareness.longitude,
    awareness_intention: awareness.awareness_intention
  }

  message = %Dartmessaging.MessageScheme{
    route: 1,  # Route for AwarenessNotification
    payload: {:awareness_notification, notification}
  }

  binary = Dartmessaging.MessageScheme.encode(message)
  send(state.ws_pid, {:binary, binary})

  {:noreply, state}
end
```

### 7.2 PingPong Example

```elixir
# Server sends PingPong REQUEST
ping = %Dartmessaging.PingPong{
  from: "server@domain.com",
  to: "client@domain.com",
  type: :REQUEST,
  status: :UNKNOWN,
  request_time: System.system_time(:millisecond)
}

message = %Dartmessaging.MessageScheme{
  route: 6, # Route for PingPong
  payload: {:pingpong_message, ping}
}

binary = Dartmessaging.MessageScheme.encode(message)
send(state.ws_pid, {:binary, binary})

# Client decodes and replies with RESPONSE
{:pingpong_message, ping_req} ->
  response = %Dartmessaging.PingPong{
    from: ping_req.to,
    to: ping_req.from,
    type: :RESPONSE,
    status: :SUCCESS,
    request_time: ping_req.request_time,
    response_time: System.system_time(:millisecond)
  }
```

### 7.3 TokenRevoke Example

```elixir
revoke_request = %Dartmessaging.TokenRevoke{
  from: %Dartmessaging.Identity{eid: "admin@domain.com", device_eid: "server1"},
  to: %Dartmessaging.Identity{eid: "user@domain.com", device_eid: "device123"},
  type: :REQUEST,
  timestamp: System.system_time(:millisecond)
}

message = %Dartmessaging.MessageScheme{
  route: 7,
  payload: {:token_revoke, revoke_request}
}

binary = Dartmessaging.MessageScheme.encode(message)
send(state.ws_pid, {:binary, binary})
```

---

## 8. Security Considerations

### Transport Security

- All BIMip communications (HTTP/WebSocket) **MUST** use TLS.

### Authentication Architecture

- User login and token issuance are handled by a separate Token Server.
- BIMip servers do **not** generate tokens; they only verify JWT tokens presented by clients.
- Tokens are signed by the Token Server using asymmetric cryptography (private key).
- BIMip servers verify tokens using the built-in public key of the Token Server.

### Token Usage on BIMip

- Clients authenticate by passing the JWT as a Bearer token in the HTTP/WebSocket headers.

**Example header:**

```http
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJhd...
```

### Token Verification (BIMip Server)

- Validate the signature, expiry, and claims (`device_eid`, `eid`) before allowing any message exchange.

---

## 9. IANA Considerations

- Introduces new namespaces `awareness`, `pingpong`, `tokenrevoke`.
- No IANA registry actions required currently.

---

## 10. References

- \[RFC 6120] Extensible Messaging and Presence Protocol (XMPP): Core, March 201
