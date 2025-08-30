# BIMip (RFC-DRAFT)

**Status:** Draft  
**Category:** Standards Track  
**Author:** Paul Aigokhai Olukayode  
**Created:** 2025-08-30

---

## Table of Contents

1. Introduction
2. Terminology
3. Message Types
   - 3.1 Awareness Messages
   - 3.2 PingPong Messages
4. Protocol Buffers Definitions
   - 4.1 Awareness
   - 4.2 PingPong
5. Semantics
   - 5.1 Awareness
   - 5.2 PingPong
6. Example Exchanges
   - 6.1 Awareness
   - 6.2 PingPong
7. Security Considerations
8. IANA Considerations
9. References

---

## 1. Introduction

The **Awareness Protocol (AWP)** defines a lightweight message-based system for communicating user and device presence ("awareness") between entities.  
It allows one entity to query the awareness of another, receive responses, and subscribe to notifications about awareness changes.

The **PingPong Protocol (PPG)** provides a standardized mechanism to verify connectivity between two entities, measure latency, and detect lost connections.

Together, they form part of **BIMip** (Binary Interface for Messaging & Internet Protocol).

---

## 2. Terminology

- **EID**: Entity Identifier (e.g., `alice@domain.com/phone`)
- **Awareness**: Presence information of an entity
- **Stanza**: A protocol message unit
- **PingPong**: Connectivity verification mechanism

---

## 3. Message Types

### 3.1 Awareness Messages

1. **AwarenessRequest** – Query another entity's awareness
2. **AwarenessResponse** – Response to a request, containing status and metadata
3. **AwarenessNotification** – Unsolicited update about awareness changes

### 3.2 PingPong Messages

1. **PingPong** – A single stanza handling both Request and Response

---

## 4. Protocol Buffers Definitions

### 4.1 Awareness

```proto
// Awareness Request
message AwarenessRequest {
  string request_id = 1;
  string from = 2;                 // Requesting entity
  string to = 3;                   // Target entity
}

// Awareness Response
message AwarenessResponse {
  string request_id = 1;           // Matches AwarenessRequest
  string from = 2;
  string to = 3;
  AwarenessStatus status = 4;      // ONLINE, OFFLINE, AWAY
  int64 last_seen = 5;             // Unix UTC timestamp
  double latitude = 6;             // Optional location
  double longitude = 7;
  AwarenessIntention intention = 8;// e.g., 1 = allow, 2 = block
}

// Awareness Notification
message AwarenessNotification {
  string from = 1;
  string to = 2;
  AwarenessStatus status = 3;
  int64 last_seen = 4;
}

// Awareness status enum
enum AwarenessStatus {
  UNKNOWN = 0;
  ONLINE = 1;
  OFFLINE = 2;
  AWAY = 3;
}

// Awareness intention enum
enum AwarenessIntention {
  UNSPECIFIED = 0;
  ALLOW = 1;
  BLOCK = 2;
}
```

---

### 4.2 PingPong

```java
// PingPong message for connection health
message PingPong {
  string from = 1;          // Sender entity (EID)
  string to = 2;            // Recipient entity (EID)
  PingType type = 3;        // REQUEST = 1, RESPONSE = 2
  PingStatus status = 4;    // UNKNOWN = 0, SUCCESS = 1, FAIL = 2
  int64 request_time = 5;   // Unix UTC timestamp of request (ms)
  int64 response_time = 6;  // Unix UTC timestamp of response (ms)
}

// Ping type: request or response
enum PingType {
  REQUEST = 1;
  RESPONSE = 2;
}

// Optional status
enum PingStatus {
  UNKNOWN = 0;
  SUCCESS = 1;
  FAIL = 2;
}

```

---

## 5. Semantics

### 5.1 Awareness

- **AwarenessRequest**: Must receive `AwarenessResponse`, unless blocked.
- **AwarenessResponse**: Must include the original `request_id`.
- **AwarenessNotification**: May be sent without acknowledgment.

### 5.2 PingPong

- **REQUEST**: Sent to verify connectivity.
- **RESPONSE**: Sent with `response_time`.
- Can be used to measure latency and detect lost connections.

---

## 6. Example Exchanges

### 6.1 Awareness

**AwarenessRequest**

```java
AwarenessRequest {
  request_id: "12345",
  from: "alice@domain.com/phone",
  to: "bob@domain.com/laptop"
}
```

### Awareness Response

```java
AwarenessResponse {
  "request_id": "12345",
  "from": "bob@domain.com/laptop",
  "to": "alice@domain.com/phone",
  "status": "ONLINE",
  "last_seen": 1693400000,
  "latitude": 6.5244,
  "longitude": 3.3792,
  "awareness_intention": 1
}

```

---

## 6.2 PingPong

### Ping Request

```java
// PingPong message for connection health
message PingPong {
  string from = 1;          // Sender entity (EID)
  string to = 2;            // Recipient entity (EID)
  PingType type = 3;        // 1 = Request, 2 = Response
  PingStatus status = 4;    // 0 = Unknown, 1 = Success, 2 = Fail
  int64 request_time = 5;   // Unix UTC timestamp of request (ms)
  int64 response_time = 6;  // Unix UTC timestamp of response (ms)
}

// Ping type: request or response
enum PingType {
  REQUEST = 1;
  RESPONSE = 2;
}

// Optional status
enum PingStatus {
  UNKNOWN = 0;
  SUCCESS = 1;
  FAIL = 2;
}

```

---

## 7. Security Considerations

- Authenticate **Awareness** requests to prevent spoofing.
- Share sensitive metadata (e.g., location) **only with authorized parties**.
- Rate-limit notifications to prevent flooding attacks.
- Use **TLS** for all transport layers to ensure confidentiality and integrity.

---

## 8. IANA Considerations

- Introduces new namespaces: **awareness** and **pingpong**.
- No IANA registry actions are required at this stage.

---

## 9. References

- [RFC 6120] _Extensible Messaging and Presence Protocol (XMPP): Core_, March 2011.
- [RFC 2778] _A Model for Presence and Instant Messaging_, February 2000.
- _Protocol Buffers Specification_.
