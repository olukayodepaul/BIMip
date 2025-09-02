Perfect! Here’s a **full RFC draft write-up for BIMip-Foundation**, updated with **all message types, semantics, and explanations** including Awareness, PingPong, TokenRevoke, Subscriber, BlockSubscriber, Logout, and Error. This is now a comprehensive single document.

---

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
   * [4.6 Logout Messages](#46-logout-messages)
   * [4.7 Error Messages](#47-error-messages)
5. [Protocol Buffers Definitions](#5-protocol-buffers-definitions)

   * [5.1 Identity](#51-identity)
   * [5.2 Awareness](#52-awareness)
   * [5.3 PingPong](#53-pingpong)
   * [5.4 TokenRevoke](#54-tokenrevoke)
   * [5.5 Subscriber](#55-subscriber)
   * [5.6 BlockSubscriber](#56-blocksubscriber)
   * [5.7 Logout](#57-logout)
   * [5.8 Error](#58-error)
   * [5.9 MessageScheme Envelope](#59-messagescheme-envelope)
6. [Semantics](#6-semantics)
7. [Example Exchanges](#7-example-exchanges)
8. [Security Considerations](#8-security-considerations)
9. [IANA Considerations](#9-iana-considerations)
10. [References](#10-references)

---

## 1. Introduction

The **Awareness Protocol (AWP)** defines a lightweight message-based system for communicating user and device presence ("awareness") between entities.

The **PingPong Protocol (PPG)** provides a mechanism to verify connectivity between devices and servers, measure latency, and detect lost connections.

The **Token Revoke Protocol (TRP)** provides JWT-based session revocation and logout management.

**Subscriber** and **Block Subscriber** protocols manage user access and communication control.

All protocols are part of **BIMip (Binary Interface for Messaging & Internet Protocol)**, optimized for internal service and mobile-backend communication.

---

## 2. Terminology

* **Epohai Identifier (EID):** Unique user identifier (e.g., `alice@domain.com`).
* **Device EID:** Identifier for a specific device under a user EID.
* **Requester:** Entity asking about awareness or connectivity.
* **Responder:** Entity providing awareness or ping response.
* **Notification:** Proactive awareness update sent without request.
* **Route:** Logical identifier in the `MessageScheme` envelope indicating the payload schema.

---

## 3. Protocol Overview

Primary message categories:

* **Awareness Messages:** Query or notify user/device awareness state.
* **PingPong Messages:** Verify device-to-server connectivity.
* **Token Revoke Messages:** Logout/revoke device sessions.
* **Subscriber Messages:** Add/remove subscribers.
* **Block Subscriber Messages:** Block unwanted communication.
* **Logout Messages:** Disconnect a device/session.
* **Error Messages:** Standardized error reporting.

All messages use **Protocol Buffers** and are wrapped in a `MessageScheme` envelope.

---

## 4. Message Types

### 4.1 Awareness Messages

* **AwarenessRequest** – Query another user/device for their current presence/status.
* **AwarenessResponse** – Reply to AwarenessRequest with status, location, and intention.
* **AwarenessNotification** – Proactively notify other users/devices of awareness state.

**Notes:** Cross-user messages; can be sent from one user to another.

---

### 4.2 PingPong Messages (Device-Level)

* **PingPong REQUEST** – Device → Server connectivity check.
* **PingPong RESPONSE** – Server → Device response with latency/status.

**Notes:** Device-to-server only; `from` is optional; `to` indicates the device/server.

---

### 4.3 Token Revoke Messages

* **TokenRevoke REQUEST** – Initiates logout for specific devices or all devices of a user.
* **TokenRevoke RESPONSE** – Confirms whether revocation succeeded.

---

### 4.4 Subscriber Messages

* **SubscriberAddRequest** – Request to add a subscriber under a user.
* **SubscriberAddResponse** – Confirms addition with status and resource ID.

---

### 4.5 Block Subscriber Messages

* **BlockSubscriber REQUEST** – Request to block a subscriber.
* **BlockSubscriber RESPONSE** – Confirms status of the block.

---

### 4.6 Logout Messages

* **Logout REQUEST** – Client/device requests logout/disconnect.
* **Logout RESPONSE** – Server confirms logout status.

**Status Codes:**

* `1 = DISCONNECT` – Client-initiated disconnect.
* `2 = FAIL` – Logout failed.
* `3 = SUCCESS` – Logout succeeded.

---

### 4.7 Error Messages

* **ErrorMessage** – Protocol-level errors (invalid requests, schema mismatches, or authorization failures).

**Fields:**

* `code` – Numeric error code.
* `message` – Human-readable short message.
* `route` – Route of the message causing the error.
* `details` – Optional detailed description.

---

## 5. Protocol Buffers Definitions

### 5.1 Identity

```proto
syntax = "proto3";
package dartmessaging;

message Identity {
  string eid = 1;                    // Unique user identifier
  string connection_resource_id = 2; // Optional: device/session binding
}
```

---

### 5.2 Awareness

```proto
message AwarenessRequest {
  Identity from = 1;
  Identity to = 2;
  int64 awareness_identifier = 3;  // Unique request ID
  int64 timestamp = 4;             // Optional Unix timestamp
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

---

### 5.3 PingPong

```proto
message PingPong {
  Identity to = 1;           // Target device or server
  int32 type = 2;            // 1=REQUEST, 2=RESPONSE
  int32 status = 3;          // 1=PENDING ... 6=UNREACHABLE
  int64 request_time = 4;
  int64 response_time = 5;
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
  int32 status = 4;       
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
  int32 status = 4;         
  string message = 5;       
  int64 timestamp = 6;
}
```

---

### 5.7 Logout

```proto
message Logout {
  Identity entity = 1;      
  int32 type = 2;           
  int32 status = 3;         
  int64 timestamp = 4;      
}
```

---

### 5.8 Error

```proto
message ErrorMessage {
  int32 code = 1;
  string message = 2;
  string route = 3;
  string details = 4;
}
```

---

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

* **Awareness:** Requests require a response; notifications may be sent proactively.
* **PingPong:** REQUEST tests connectivity; RESPONSE echoes timestamps/status.
* **TokenRevoke:** Servers must invalidate sessions immediately.
* **Subscriber:** Responses must echo `subscriber_resource_id`.
* **BlockSubscriber:** REQUEST indicates block; RESPONSE confirms status.
* **Logout:** REQUEST is client/device-initiated; RESPONSE confirms success/failure.
* **Error:** Can be returned in response to any message type.

---

## 7. Example Exchanges

### Awareness Notification

```elixir
notification = %Dartmessaging.AwarenessNotification{
  from: %Dartmessaging.Identity{eid: "alice@domain.com"},
  to: %Dartmessaging.Identity{eid: "bob@domain.com"},
  last_seen: System.system_time(:millisecond),
  status: 1
}

message = %Dartmessaging.MessageScheme{
  route: 1,
  payload: {:awareness_notification, notification}
}
```

### PingPong Request

```elixir
ping = %Dartmessaging.PingPong{
  to: %Dartmessaging.Identity{eid: "client@domain.com", connection_resource_id: "NJBCHIBASJBKASJJCNAN"},
  type: 1,  # REQUEST
  status: 1,
  request_time: System.system_time(:millisecond)
}
```

### Token Revoke Request

```elixir
revoke_request = %Dartmessaging.TokenRevokeRequest{
  to: %Dartmessaging.Identity{eid: "client@domain.com", connection_resource_id: "NJBCHIBASJBKASJJCNAN"},
  token: "jwt-token-string",
  timestamp: System.system_time(:millisecond)
}
```

### Subscriber Add

```elixir
subscriber_request = %Dartmessaging.SubscriberAddRequest{
  owner: %Dartmessaging.Identity{eid: "alice@domain.com"},
  subscriber: %Dartmessaging.Identity{eid: "bob@domain.com"},
  nickname: "Bobby",
  group: "Friends",
  subscriber_resource_id: "sub-123",
  timestamp: System.system_time(:millisecond)
}
```

### Block Subscriber

```elixir
block_request = %Dartmessaging.BlockSubscriber{
  owner: %Dartmessaging.Identity{eid: "alice@domain.com"},
  subscriber: %Dartmessaging.Identity{eid: "bob@domain.com"},
  type: 1,  # REQUEST
  status: 0,
  message: "Block this subscriber",
  timestamp: System.system_time(:millisecond)
}
```

### Logout

```elixir
logout_req = %Dartmessaging.Logout{
  entity: %Dartmessaging.Identity{eid: "user@domain.com"},
  type: 1,
  status: 1,
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

* Introduces new namespaces: awareness, pingpong, subscriber, logout.
* No IANA registry actions required currently.

---

## 10. References

* \[RFC 6120] Extensible Messaging and Presence Protocol (XMPP): Core, March 2011
* \[RFC 2778] Instant Messaging / Presence Protocol Requirements, February 2000

---

This draft is now **fully comprehensive**. It captures:

* Device vs. user-level semantics (PingPong vs. Awareness).
* All message types including Logout and Error.
* Protocol Buffers definitions ready for implementation.
* Example exchanges for clarity.


