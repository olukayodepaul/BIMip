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
   - [4.4 Contact Messages](#44-contact-messages)
   - [4.5 Block Subscriber Messages](#45-block-subscriber-messages)
5. [Protocol Buffers Definitions](#5-protocol-buffers-definitions)
   - [5.1 Identity](#51-identity)
   - [5.2 Awareness](#52-awareness)
   - [5.3 PingPong](#53-pingpong)
   - [5.4 TokenRevoke](#54-tokenrevoke)
   - [5.5 Contact](#55-contact)
   - [5.6 BlockSubscriber](#56-blocksubscriber)
   - [5.7 Error](#57-error)
   - [5.8 MessageScheme Envelope](#58-messagescheme-envelope)
6. [Semantics](#6-semantics)
7. [Example Exchanges](#7-example-exchanges)
8. [Security Considerations](#8-security-considerations)
9. [IANA Considerations](#9-iana-considerations)
10. [References](#10-references)

---

## 1. Introduction

The Awareness Protocol (AWP) defines a lightweight message-based system for communicating user and device presence ("awareness") between entities.  

The PingPong Protocol (PPG) provides a standardized mechanism to verify connectivity between entities, measure latency, and detect lost connections.

The Token Revoke Protocol (TRP) defines a mechanism for logging out specific devices or all devices of a user using JWT-based authentication.

Contact and Block protocols allow managing contacts and blocking unwanted subscribers.

Together, these form part of **BIMip (Binary Interface for Messaging & Internet Protocol).**

---

## 2. Terminology

- **Epohai Identifier (EID):** Unique identifier for a user, e.g., `alice@domain.com`.  
- **Device EID:** Identifier for a specific device under a user EID.  
- **Requester:** Entity asking about awareness or connectivity.  
- **Responder:** Entity providing awareness or ping response.  
- **Notification:** Proactive awareness update sent without request.  
- **Route:** Logical identifier in the wrapper indicating which payload schema is carried.  

---

## 3. Protocol Overview

Primary message categories:

- **Awareness Messages:** Query or notify awareness state.  
- **PingPong Messages:** Verify connectivity and latency.  
- **Token Revoke Messages:** Revoke JWT-based sessions.  
- **Contact Messages:** Add/remove contacts.  
- **Block Subscriber Messages:** Block a subscriber from sending messages or notifications.  

All messages are encoded using **Protocol Buffers** and wrapped in a `MessageScheme` envelope containing a `route` and a `oneof payload`.

---

## 4. Message Types

### 4.1 Awareness Messages

- `AwarenessRequest` – Query awareness state.  
- `AwarenessResponse` – Return awareness state.  
- `AwarenessNotification` – Proactive updates.

### 4.2 PingPong Messages

- `PingPong REQUEST` – Verify connectivity.  
- `PingPong RESPONSE` – Reply indicating success/failure.  

### 4.3 Token Revoke Messages

- `TokenRevoke REQUEST` – Initiates logout.  
- `TokenRevoke RESPONSE` – Confirms revocation.  

### 4.4 Contact Messages

- `ContactAddRequest` – Add a contact.  
- `ContactAddResponse` – Response to addition.  

### 4.5 Block Subscriber Messages

- `BlockSubscriber` – Request/response to block a subscriber.  

---

## 5. Protocol Buffers Definitions

### 5.1 Identity

```proto
syntax = "proto3";
package dartmessaging;

message Identity {
  string eid = 1;                     // Unique user/entity identifier
  string connection_resource_id = 2;  // Optional: device/session binding
}


---

### 5.2 Awareness

```proto
message AwarenessRequest {
  Identity from = 1;
  Identity to = 2;
  int64 awareness_identifier = 3;  // Unique request ID
  int64 timestamp = 4;
}

message AwarenessResponse {
  Identity from = 1;
  Identity to = 2;
  string awareness_identifier = 3;  // Must match request
  int32 status = 4;                 // 1=ONLINE ... 8=UNKNOWN
  double latitude = 5;              // Optional
  double longitude = 6;             // Optional
  int32 awareness_intention = 7;    // Optional
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

---

### 5.3 PingPong

```proto
message PingPong {
  Identity from = 1;
  Identity to = 2;
  int32 type = 3;          // 1=REQUEST, 2=RESPONSE
  int32 status = 4;        // 1=PENDING ... 6=UNREACHABLE
  int64 request_time = 5;
  int64 response_time = 6;
}
```

---

### 5.4 TokenRevoke

```proto
message TokenRevokeRequest {
  Identity to = 1;
  string token = 2;       // JWT string
  int64 timestamp = 3;
}

message TokenRevokeResponse {
  Identity to = 1;
  int32 status = 2;       // 1=SUCCESS, 2=FAILED
  int64 timestamp = 3;
}
```

---

### 5.5 Contact

```proto
message ContactAddRequest {
  Identity owner = 1;
  Identity subscriber = 2;
  string nickname = 3;
  string group = 4;
  string contact_resource_id = 5;
  int64 timestamp = 6;
}

message ContactAddResponse {
  Identity owner = 1;
  Identity subscriber = 2;
  string contact_resource_id = 3;
  int32 status = 4;       // 1=SUCCESS, 2=FAILED
  string message = 5;
  int64 timestamp = 6;
}
```

---

### 5.6 BlockSubscriber

```proto
message BlockSubscriber {
  Identity owner = 1;       
  Identity subscriber = 2;  
  int32 type = 3;           // 1=REQUEST, 2=RESPONSE
  int32 status = 4;         // 1=SUCCESS, 2=FAILED (if RESPONSE)
  string message = 5;       
  int64 timestamp = 6;
}
```

---

### 5.7 Error

```proto
message ErrorMessage {
  int32 code = 1;
  string message = 2;
  string route = 3;
  string details = 4;
}
```

---

### 5.8 MessageScheme Envelope

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
    ContactAddRequest contact_add_request = 9;
    ContactAddResponse contact_add_response = 10;
    BlockSubscriber block_subscriber = 11;
  }
}
```

---

## 6. Semantics

* **Awareness:** Requests must be answered; notifications may be sent proactively.
* **PingPong:** REQUEST tests connectivity; RESPONSE echoes timestamps and status.
* **TokenRevoke:** Servers must invalidate sessions immediately.
* **Contact:** Responses must echo `contact_resource_id`.
* **BlockSubscriber:** REQUEST indicates block; RESPONSE confirms status.

---

## 7. Example Exchanges

### 7.1 Awareness

```elixir
notification = %Dartmessaging.AwarenessNotification{
  from: "alice@domain.com",
  last_seen: DateTime.to_unix(awareness.last_seen, :second),
  status: awareness.status
}

message = %Dartmessaging.MessageScheme{
  route: 1,
  payload: {:awareness_notification, notification}
}
```

### 7.2 PingPong

```elixir
ping = %Dartmessaging.PingPong{
  from: "server@domain.com",
  to: "client@domain.com",
  type: 1,  # REQUEST
  status: 1,
  request_time: System.system_time(:millisecond)
}
```

### 7.3 TokenRevoke

```elixir
revoke_request = %Dartmessaging.TokenRevokeRequest{
  to: %Dartmessaging.Identity{eid: "user@domain.com"},
  token: "jwt-token-string",
  timestamp: System.system_time(:millisecond)
}
``
```
