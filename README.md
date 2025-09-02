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

   * [4.1 Awareness Messages](#41-awareness-messages)
   * [4.2 PingPong Messages](#42-pingpong-messages)
   * [4.3 Token Revoke Messages](#43-token-revoke-messages)
   * [4.4 Subscriber Messages](#44-subscriber-messages)
   * [4.5 Block Subscriber Messages](#45-block-subscriber-messages)
5. [Protocol Buffers Definitions](#5-protocol-buffers-definitions)

   * [5.1 Identity](#51-identity)
   * [5.2 Awareness](#52-awareness)
   * [5.3 PingPong](#53-pingpong)
   * [5.4 TokenRevoke](#54-tokenrevoke)
   * [5.5 Subscriber](#55-subscriber)
   * [5.6 BlockSubscriber](#56-blocksubscriber)
   * [5.7 Error](#57-error)
   * [5.8 MessageScheme Envelope](#58-messagescheme-envelope)
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

Subscriber and Block protocols allow managing subscribers and blocking unwanted messages.

Together, these form part of **BIMip (Binary Interface for Messaging & Internet Protocol).**

---

## 2. Terminology

* **Epohai Identifier (EID):** Unique identifier for a user, e.g., `alice@domain.com`.
* **Device EID:** Identifier for a specific device under a user EID.
* **Requester:** Entity asking about awareness or connectivity.
* **Responder:** Entity providing awareness or ping response.
* **Notification:** Proactive awareness update sent without request.
* **Route:** Logical identifier in the wrapper indicating which payload schema is carried.

---

## 3. Protocol Overview

Primary message categories:

* **Awareness Messages:** Query or notify awareness state.
* **PingPong Messages:** Verify connectivity and latency.
* **Token Revoke Messages:** Revoke JWT-based sessions.
* **Subscriber Messages:** Add/remove subscribers.
* **Block Subscriber Messages:** Block a subscriber from sending messages or notifications.

All messages are encoded using **Protocol Buffers** and wrapped in a `MessageScheme` envelope containing a `route` and a `oneof payload`.

---

## 4. Message Types

### 4.1 Awareness Messages

* `AwarenessRequest` – Query awareness state.
* `AwarenessResponse` – Return awareness state.
* `AwarenessNotification` – Proactive updates.

### 4.2 PingPong Messages

* `PingPong REQUEST` – Verify connectivity.
* `PingPong RESPONSE` – Reply indicating success/failure.

### 4.3 Token Revoke Messages

* `TokenRevoke REQUEST` – Initiates logout.
* `TokenRevoke RESPONSE` – Confirms revocation.

### 4.4 Subscriber Messages

* `SubscriberAddRequest` – Add a subscriber.
* `SubscriberAddResponse` – Response to addition.

### 4.5 Block Subscriber Messages

* `BlockSubscriber` – Request/response to block a subscriber.

---

## 5. Protocol Buffers Definitions

### 5.1 Identity

```proto
syntax = "proto3";
package dartmessaging;


```

---

### 5.2 Awareness

```proto
message AwarenessRequest = %Dartmessaging.AwarenessRequest{
  from: %Dartmessaging.Identity{ eid: "a@domain.com" },
  to: %Dartmessaging.Identity{ eid: "b@domain.com" },
  awareness_identifier: System.system_time(:millisecond),
  timestamp: 0
}


message AwarenessResponse {
  from: %Dartmessaging.Identity{ eid: "a@domain.com" },
  to: %Dartmessaging.Identity{ eid: "b@domain.com" },
  string awareness_identifier = 3;  
  int32 status = 4;                 // 1=ONLINE, 2=OFFLINE, 3=AWAY, 4=BUSY, 5=DO_NOT_DISTURB, 6=INVISIBLE, 7=IDLE, 8=UNKNOWN
  double latitude = 5;              
  double longitude = 6;             
  int32 awareness_intention = 7;    
  int64 timestamp = 8;
}

message AwarenessNotification {
  Identity from = 1;
  Identity to = 2;
  int32 status = 3;                 // 1=ONLINE, 2=OFFLINE, 3=AWAY, 4=BUSY, 5=DO_NOT_DISTURB, 6=INVISIBLE, 7=IDLE, 8=UNKNOWN
  int64 last_seen = 4;
  double latitude = 5;
  double longitude = 6;
  int32 awareness_intention = 7;
}
```

**Awareness Status Codes:**

| Code | Status           | Meaning                             |
| ---- | ---------------- | ----------------------------------- |
| 1    | ONLINE           | User/device is online and available |
| 2    | OFFLINE          | User/device is offline              |
| 3    | AWAY             | Temporarily away from device        |
| 4    | BUSY             | Busy, do not disturb                |
| 5    | DO\_NOT\_DISTURB | Explicit DND state                  |
| 6    | INVISIBLE        | Online but appears offline          |
| 7    | IDLE             | Online but inactive for a period    |
| 8    | UNKNOWN          | Unknown or undetermined state       |

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
  string token = 2;       
  int64 timestamp = 3;
}

message TokenRevokeResponse {
  Identity to = 1;
  int32 status = 2;       // 1=SUCCESS, 2=FAILED
  int64 timestamp = 3;
}
```

---

### 5.5 Subscriber

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


### 5.8 Logout


```proto
// Logout action
message Logout {
  Identity entity = 1;      // The user/device performing logout
  int32 type = 2;           // 1 = REQUEST, 2 = RESPONSE
  int32 status = 3;         // 1 = DISCONNECT, 2 = FAIL, 3 = SUCCESS
  int64 timestamp = 4;      // Unix UTC timestamp (ms) of the action
}
```

**Notes:**

* `entity` uses **Identity** (eid + optional connection\_resource\_id)
* `type` distinguishes **REQUEST** (client wants to logout) from **RESPONSE** (server confirms or fails)
* `status` shows the result of the logout attempt
* `timestamp` ensures auditable/loggable action


### Example Usage

```elixir
# Client initiates logout request
logout_req = %Dartmessaging.Logout{
  entity: %Dartmessaging.Identity{eid: "user@domain.com", connection_resource_id: "device123"},
  type: 1,    # REQUEST
  status: 1,  # DISCONNECT
  timestamp: System.system_time(:millisecond)
}

message = %Dartmessaging.MessageScheme{
  route: 12,  # new route for Logout
  payload: {:logout, logout_req}
}

binary = Dartmessaging.MessageScheme.encode(message)
send(state.ws_pid, {:binary, binary})
```

```elixir
# Server responds
logout_resp = %Dartmessaging.Logout{
  entity: %Dartmessaging.Identity{eid: "user@domain.com", connection_resource_id: "device123"},
  type: 2,    # RESPONSE
  status: 3,  # SUCCESS
  timestamp: System.system_time(:millisecond)
}

message = %Dartmessaging.MessageScheme{
  route: 12,
  payload: {:logout, logout_resp}
}

binary = Dartmessaging.MessageScheme.encode(message)
send(state.ws_pid, {:binary, binary})
```

### 5.9 MessageScheme Envelope

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
  }
}
```


---

## 6. Semantics

* **Awareness:** Requests must be answered; notifications may be sent proactively.
* **PingPong:** REQUEST tests connectivity; RESPONSE echoes timestamps and status.
* **TokenRevoke:** Servers must invalidate sessions immediately.
* **Subscriber:** Responses must echo `subscriber_resource_id`.
* **BlockSubscriber:** REQUEST indicates block; RESPONSE confirms status.

---

## 7. Example Exchanges

### 7.1 Awareness

```elixir
notification = %Dartmessaging.AwarenessNotification{
  from: %Dartmessaging.Identity{eid: "alice@domain.com"},
  to: %Dartmessaging.Identity{eid: "bob@domain.com"},
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
  from: %Dartmessaging.Identity{eid: "server@domain.com"},
  to: %Dartmessaging.Identity{eid: "client@domain.com"},
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
```

### 7.4 Subscriber Add

```elixir
subscriber_request = %Dartmessaging.SubscriberAddRequest{
  owner: %Dartmessaging.Identity{eid: "alice@domain.com"},
  subscriber: %Dartmessaging.Identity{eid: "bob@domain.com", connection_resource_id: "conn-01"},
  nickname: "Bobby",
  group: "Friends",
  subscriber_resource_id: "sub-123",
  timestamp: System.system_time(:millisecond)
}

subscriber_response = %Dartmessaging.SubscriberAddResponse{
  owner: %Dartmessaging.Identity{eid: "alice@domain.com"},
  subscriber: %Dartmessaging.Identity{eid: "bob@domain.com", connection_resource_id: "conn-01"},
  subscriber_resource_id: "sub-123",
  status: 1,  # SUCCESS
  message: "Subscriber added successfully",
  timestamp: System.system_time(:millisecond)
}
```

### 7.5 Block Subscriber

```elixir
block_request = %Dartmessaging.BlockSubscriber{
  owner: %Dartmessaging.Identity{eid: "alice@domain.com"},
  subscriber: %Dartmessaging.Identity{eid: "bob@domain.com"},
  type: 1,  # REQUEST
  status: 0,
  message: "Block this subscriber",
  timestamp: System.system_time(:millisecond)
}

block_response = %Dartmessaging.BlockSubscriber{
  owner: %Dartmessaging.Identity{eid: "alice@domain.com"},
  subscriber: %Dartmessaging.Identity{eid: "bob@domain.com"},
  type: 2,  # RESPONSE
  status: 1, # SUCCESS
  message: "Subscriber blocked",
  timestamp: System.system_time(:millisecond)
}
```

---

## 8. Security Considerations

* Authenticate awareness requests to prevent spoofing.
* Share sensitive metadata (e.g., location) only with authorized parties.
* Rate-limit notifications to prevent flooding.
* Use TLS for all transport layers.

---

## 9. IANA Considerations

* Introduces new namespaces: awareness, pingpong, subscriber.
* No IANA registry actions required currently.

---

## 10. References

* \[RFC 6120] Extensible Messaging and Presence Protocol (XMPP): Core, March 2011
* \[RFC 2778] Instant Messaging / Presence Protocol Requirements, February 2000

---

This is now a **full RFC draft** with **Awareness status codes fully enumerated**, consistent **Subscriber/Block naming**, and Identity usage for `eid` and `connection_resource_id`.

---

If you want, the **next step could be adding Login/Logout messages** with status codes like `DISCONNECT`, `FAIL`, etc., in the same consistent style.

Do you want me to add that now?
