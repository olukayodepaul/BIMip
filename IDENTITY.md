## 🔑 Identity

The `Identity` message defines a fully addressable entity within the system, allowing messages and signals to be routed to a user's account, a specific device/session, or a hosting server node.

### Protobuf Definition

```protobuf
message Identity {
  string eid = 1;       
  optional string connection_resource_id = 2; 
  optional string node = 3;  
}
```

### Field Reference

| Tag | Field Name | Type | Description | Purpose in Routing |
| :---: | :--- | :--- | :--- | :--- |
| **1** | `eid` | `string` | **Entity Identifier**. The mandatory, globally unique ID for the user or logical entity (e.g., account name, JID). | **User-level addressing.** |
| **2** | `connection_resource_id` | `optional string` | **Device/Session ID**. A unique identifier for a specific device or connection session (e.g., a mobile client instance). | **Device-specific routing.** |
| **3** | `node` | `optional string` | **Server Node ID**. The identifier for the system node or cluster instance currently managing the entity's connection state. | **Internal server routing.** |

