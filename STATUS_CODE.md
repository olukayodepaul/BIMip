## ✅ 2xx: Success (Delivery and Processing)

These codes indicate that the requested action (delivery, read, etc.) was successfully executed.

| Code | Name | Description | Used By |
| :---: | :--- | :--- | :--- |
| **200** | **OK** | The message was successfully **delivered** to the recipient's device. | Server/Recipient |
| **201** | **PROCESSED** | The message payload (e.g., a control signal or key exchange) was successfully processed by the receiving device. | Recipient |
| **202** | **VIEWED/READ** | The message was successfully opened and **viewed** by the recipient. | Recipient |

---

## 🧭 3xx: Redirection/Status Update

These codes indicate an update about the status or location of the message/recipient, though less common in pure messaging ACKs.

| Code | Name | Description | Used By |
| :---: | :--- | :--- | :--- |
| **301** | **MOVED** | The recipient's primary connection or route has permanently changed. | Server |
| **304** | **QUEUED** | The message was accepted and placed in the recipient's **offline queue** but not yet delivered to a device. | Server |

---

## ⚠️ 4xx: Client Error (Sender/Recipient Issue)

These codes indicate an error due to the message structure or the recipient's state.

| Code | Name | Description | Used By |
| :---: | :--- | :--- | :--- |
| **400** | **BAD REQUEST** | The `Message` structure was malformed (e.g., missing required fields, invalid Protobuf syntax). | Server |
| **401** | **UNAUTHORIZED** | The message signature failed **authentication/integrity checks**. | Server/Recipient |
| **403** | **BLOCKED** | The sender is blocked by the recipient, or the recipient is unknown. | Server |
| **404** | **NOT FOUND** | The recipient's primary identity (`eid`) is unknown to the system. | Server |
| **408** | **TIMEOUT** | Delivery attempt timed out; the recipient device was unresponsive or disconnected for too long. | Server |
| **409** | **OUT OF ORDER** | The message was received out of sequence relative to the expected stream offset. | Recipient |
| **410** | **INVALID ENCRYPTION** | Failed to decrypt the message, or the specified `encryption_type` is unsupported. | Recipient |

---

## ❌ 5xx: Server Error (System Issue)

These codes indicate a failure on the server side to process or deliver the message.

| Code | Name | Description | Used By |
| :---: | :--- | :--- | :--- |
| **500** | **INTERNAL ERROR** | A general, unexpected failure occurred on the server while attempting to process the message or ACK. | Server |
| **503** | **SERVICE UNAVAILABLE** | The server or necessary downstream service (e.g., storage) is temporarily overloaded or down. | Server |
