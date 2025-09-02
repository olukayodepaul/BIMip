
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
7. [Status & Type Tables](#7-status--type-tables)
8. [Example Exchanges](#8-example-exchanges)
9. [Security Considerations](#9-security-considerations)
10. [IANA Considerations](#10-iana-considerations)
11. [References](#11-references)

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

*(Same as previous write-up: Identity, Awareness, PingPong, TokenRevoke, Subscriber, BlockSubscriber, Logout, Error, MessageScheme envelope)*

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

## 7. Status & Type Tables

### PingPong

| Field  | Code | Meaning     |
| ------ | ---- | ----------- |
| type   | 1    | REQUEST     |
| type   | 2    | RESPONSE    |
| status | 1    | PENDING     |
| status | 2    | OK          |
| status | 3    | FAILED      |
| status | 4    | TIMEOUT     |
| status | 5    | UNKNOWN     |
| status | 6    | UNREACHABLE |

### Logout

| Field  | Code | Meaning    |
| ------ | ---- | ---------- |
| status | 1    | DISCONNECT |
| status | 2    | FAIL       |
| status | 3    | SUCCESS    |
| type   | 1    | REQUEST    |
| type   | 2    | RESPONSE   |

### Subscriber / BlockSubscriber

| Field  | Code | Meaning  |
| ------ | ---- | -------- |
| status | 0    | PENDING  |
| status | 1    | SUCCESS  |
| status | 2    | FAILED   |
| type   | 1    | REQUEST  |
| type   | 2    | RESPONSE |

### TokenRevoke

| Field  | Code | Meaning  |
| ------ | ---- | -------- |
| status | 1    | SUCCESS  |
| status | 2    | FAILED   |
| type   | 1    | REQUEST  |
| type   | 2    | RESPONSE |

### Awareness

| Field                | Code | Meaning   |
| -------------------- | ---- | --------- |
| status               | 0    | OFFLINE   |
| status               | 1    | ONLINE    |
| status               | 2    | AWAY      |
| status               | 3    | BUSY      |
| status               | 4    | UNKNOWN   |
| awareness\_intention | 0    | NONE      |
| awareness\_intention | 1    | AVAILABLE |
| awareness\_intention | 2    | BUSY      |
| awareness\_intention | 3    | AWAY      |

---

## 8. Example Exchanges

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

### PingPong Request with RTT

```elixir
request_time = System.system_time(:millisecond)

ping = %Dartmessaging.PingPong{
  to: %Dartmessaging.Identity{
    eid: "client@domain.com",
    connection_resource_id: "NJBCHIBASJBKASJJCNAN"
  },
  type: 1,  # REQUEST
  status: 1, # PENDING
  request_time: request_time
}

# Example server response
response_time = System.system_time(:millisecond)

ping_response = %Dartmessaging.PingPong{
  to: ping.to,
  type: 2,   # RESPONSE
  status: 2, # OK
  request_time: ping.request_time,
  response_time: response_time
}

rtt = ping_response.response_time - ping_response.request_time
```

### Token Revoke Request

```elixir
revoke_request = %Dartmessaging.TokenRevokeRequest{
  to: %Dartmessaging.Identity{
    eid: "client@domain.com",
    connection_resource_id: "NJBCHIBASJBKASJJCNAN"
  },
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

## 9. Security Considerations

* Authenticate awareness requests to prevent spoofing.
* Share sensitive metadata (e.g., location) only with authorized parties.
* Rate-limit notifications to prevent flooding.
* Use TLS for all transport layers.

---

## 10. IANA Considerations

* Introduces new namespaces: awareness, pingpong, subscriber, logout.
* No IANA registry actions required currently.

---

## 11. References

* \[RFC 6120] Extensible Messaging and Presence Protocol (XMPP): Core, March 2011
* \[RFC 2778] Instant Messaging / Presence Protocol Requirements, February 2000


