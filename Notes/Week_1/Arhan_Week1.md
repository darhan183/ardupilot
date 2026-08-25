### WEEK 1 - Drone Subsystems, Environment, Build System and SITL

##### Prepared by: Arhan Dayal

###### Date: 25/08/26

----

##### <u>Block Diagram of Multirotor</u>:

<img src="file:///home/d.arhan/Pictures/Screenshots/2026-08-25-14-46-28-image.png" title="" alt="" width="938">

---

##### <u>MAVLink</u>:

- ****What is MAVLink ?**** [[MAVLink Developer Guide | MAVLink Guide](https://mavlink.io/en/) & [MAVLink Basics — Dev documentation](https://ardupilot.org/dev/docs/mavlink-basics.html)]
  
  - MAVLink = `Micro Air Vehicle Link`
  
  - MAVLink is a lightweight messaging protocol for communicating with drones.
  
  - NAVLink follows modern hybrid `publish-subscribe` & `point to point` design pattern.
    
    - Data Streams are sent/published as topics while configuration sub-protocols such as `mission-protocol` or `parameter protocol` are `point to point` with retransmission.
  
  - MAVLink messages can be sent over almost any serial connection.
  
  - Messages are defined within `XML files`.
    
    - Each `XML files` defines `message set` supported by particular MAVLink system also referred to as `Dialect`.
    
    - The reference message set that is implemented by `Ground Control Stations` (GCS) & autopilot is defined in `common.xml`.

---

### <u>MAVLink Versions</u>

- Currently `MAVLink Version = MAVLink 2.0`, also supports `MAVLink 1.0`.

- ****<u>Determining Protocol/Message Version</u>:****
  
  - The major version can be determined from the packet start marker byte:
    
    - MAVLink 1: `0xFE`
    - MAVLink 2: `0xFD`

---

### <u>MAVLink 2</u>

- **What is MAVLink ?** 
  
  - MAVLink = `Micro Air Vehicle Link`
  
  - MAVLink is a lightweight messaging protocol for communicating with drones.
  
  - NAVLink follows modern hybrid `publish-subscribe` & `point to point` design pattern.
    
    - Data Streams are sent/published as topics while configuration sub-protocols such as `mission-protocol` or `parameter protocol` are `point to point` with retransmission.
  
  - MAVLink messages can be sent over almost any serial connection.
  
  - Messages are defined within `XML files`.
    
    - Each `XML files` defines `message set` supported by particular MAVLink system also referred to as `Dialect`.
    
    - The reference message set that is implemented by `Ground Control Stations` (GCS) & autopilot is defined in `common.xml`.

---

##### <u>Packet Serialization</u>

- **<u>MAVLink 2 Packet Format</u>:**
  
  <img src="file:///home/d.arhan/Pictures/Screenshots/2026-08-21-21-35-34-image.png" title="" alt="" width="918">

| Byte Index                                                 | C version                | Content                              | Value        | Explanation                                                                                                                                                             |
| ---------------------------------------------------------- | ------------------------ | ------------------------------------ | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0                                                          | uint8_t magic            | Packet start marker                  | 0xFD         | Protocol-specific start-of-text (STX) marker used to indicate the beginning of a new packet. Any system that does not understand protocol version will skip the packet. |
| 1                                                          | uint8_t len              | Payload length                       | 0 - 255      | Indicates length of the following payload section. This may be affected by payload truncation.                                                                          |
| 2                                                          | uint8_t incompat_flags   | Incompatibility Flags                |              | Flags that must be understood for MAVLink compatibility (implementation discards packet if it does not understand flag).                                                |
| 3                                                          | uint8_t compat_flags     | Compatibility Flags                  |              | Flags that can be ignored if not understood (implementation can still handle packet even if it does not understand flag).                                               |
| 4                                                          | uint8_t seq              | Packet sequence number               | 0 - 255      | Used to detect packet loss. Components increment value for each message sent.                                                                                           |
| 5                                                          | uint8_t sysid            | System ID (sender)                   | 1 - 255      | ID of system (vehicle) sending the message. Used to differentiate systems on network.                                                                                   |
| 6                                                          | uint8_t compid           | Component ID (sender)                | 1 - 255      | ID of component sending the message. Used to differentiate components in a system (e.g. autopilot and a camera).                                                        |
| 7 to 9                                                     | uint32_t msgid:24        | Message ID (low, middle, high bytes) | 0 - 16777215 | ID of message type in payload. Used to decode data back into message object.                                                                                            |
| For n-byte payload:<br>n=0: NA, n=1: 10, n>=2: 10 to (9+n) | uint8_t payload[max 255] | Payload                              |              | Message data. Depends on message type (i.e. Message ID) and contents.                                                                                                   |
| (n+10) to (n+11)                                           | uint16_t checksum        | Checksum(low byte, high byte))       |              |                                                                                                                                                                         |
| (n+12) to (n+24)                                           | uint8_t signature[13]    | Signature                            |              | Signature to ensure the link is tamper-proof.                                                                                                                           |

- The minimum packet length is 12 bytes for acknowledgment packets without payload.

- The maximum packet length is 280 bytes for a signed message that uses the whole payload.

- ****<u>MAVLink 1 Packet Format</u>:****
  
  <img src="file:///home/d.arhan/Pictures/Screenshots/2026-08-24-13-29-44-image.png" title="" alt="2026-08-24-13-29-44-image.png" width="637">
  
  | Byte Index                                               | C version                | Content                        | Value   | Explanation                                                                                                                                                             |
  | -------------------------------------------------------- | ------------------------ | ------------------------------ | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | 0                                                        | uint8_t magic            | Packet start marker            | 0xFE    | Protocol-specific start-of-text (STX) marker used to indicate the beginning of a new packet. Any system that does not understand protocol version will skip the packet. |
  | 1                                                        | uint8_t len              | Payload length                 | 0 - 255 | Indicates length of the following payload section (fixed for a particular message).                                                                                     |
  | 2                                                        | uint8_t seq              | Packet sequence number         | 0 - 255 | Used to detect packet loss. Components increment value for each message sent.                                                                                           |
  | 3                                                        | uint8_t sysid            | System ID                      | 1 - 255 | ID of system (vehicle) sending the message. Used to differentiate systems on network.                                                                                   |
  | 4                                                        | uint8_t compid           | Component ID                   | 1 - 255 | ID of component sending the message. Used to differentiate components in a system (e.g. autopilot and a camera).                                                        |
  | 5                                                        | uint8_t msgid            | Message ID                     | 0 - 255 | ID of message type in payload. Used to decode data back into message object.                                                                                            |
  | For n-byte payload:<br>n=0: NA, n=1: 6, n>=2: 6 to (5+n) | uint8_t payload[max 255] | Payload data                   |         | Message data. Content depends on message type (i.e. Message ID).                                                                                                        |
  | (n+6) to (n+7)                                           | uint16_t checksum        | Checksum (low byte, high byte) |         |                                                                                                                                                                         |
  
  - The minimum packet length is 8 bytes for acknowledgment packets without payload.
  
  - The maximum packet length is 263 bytes for full payload.

- **<u>Payload Format</u>:** Messages are encoded within the MAVLink packet:
  
  - The `msgid` (message id) field identifies the specific message encoded in the packet.
  
  - The `payload` field contains the message data.
    
    - MAVLink reorders the message fields in the payload for over-the-wire transmission.
    
    - MAVLink 2 `truncates` any zero-filled bytes at the end of the payload before the message is sent and sets the packet `len` field appropriately.
  
  - The `len` field contains the length of the payload data.

- **<u>Field Reordering</u>:** Message payload fields are reordered for transmission as follows:
  
  - Fields are sorted according to their native data size:
    
    - `uint64_t`, `double` - 8 bytes
    - `uint32_t`, `float`  - 4 bytes
    - `uint16_t`  - 2 bytes
    - `uint8_t`, `char`  - 1 bytes
  
  - If two fields have the same length, their order is preserved as it was present before the data field size ordering.
  
  - Arrays are handled based on the data type they use, not based on the total array size.

- ****<u>Empty-Byte Payload Truncation(MAVLink 2)</u>:**** `MAVLink 2` truncate any empty (zero-filled) bytes at the end of the serialized payload before it is sent. This contrasts with `MAVLink 1`, where bytes were sent for all fields regardless of content.

---

### <u>DroneCAN</u>

- is a lightweight protocol designed to provide a highly reliable communication method for robotic application via `CAN BUS`.

- The DroneCAN network is a decentralized peer network, where each peer (node) has a unique numeric identifier-`node ID`

- The nodes of DroneCAN network can communicate using any of the following communication methods:
  
  - ****<u>Message Broadcasting</u>****: The primary method of data exchange with publish/subscribe methods.
  
  - ****<u>Service Invocation</u>****: The communication method for peer-to-peer request/response interactions.

- A predefined set of data structures is used, where each data structure has a unique identifier - the `Data Type ID (DTID)`.

- Some data structures are standard and defined by the protocol specification; others may be specific to a particular application.

- A pair of `data type ID` and `node ID` can be used to support redundant nodes with identical functionality inside the same network.

- Message and service data structures are defined using the `Data Structure Description Language (DSDL)`.

#### <u>Message Broadcasting</u>:

- It refers to the transmission of a serialized data structure over CAN bus to other nodes.

- This is the primary DroneCAN data exchange mechanism.

- Typical use cases may include transfer of the following kinds of data: **sensor measurements, actuator commands, or equipment status information.**

- A broadcast message includes the following information:
  
  | Field          | Content                                                                                                   |
  | -------------- | --------------------------------------------------------------------------------------------------------- |
  | Payload        | The serialized data structure                                                                             |
  | Data type ID   | Numerical identifier that indicates how the data structure should be interpreted                          |
  | Source node ID | The node ID of the transmitting node                                                                      |
  | Transfer ID    | A small overflowing integer that increments with every transfer of this type of message from a given node |

- ##### Anonymous message broadcasting:
  
  - Nodes that don’t have a unique node ID can publish anonymous messages.
  
  - An anonymous message is different from a regular message in that it doesn’t contain source node ID.
  
  - This kind of data exchange is useful during initial configuration of the node, particularly during dynamic node ID allocation procedure.

#### <u>CAN Bus Transport Layer</u>:

- ****The Concept of Transfer:**** A Transfer is an act of data transmission between nodes.
  
  - A transfer that is addressed to one particular node is a `unicast transfer`.
  
  - A transfer that is addressed to all nodes except the source node is a `broadcast transfer`.
  
  - DroneCAN defines the following types of transfers:
    
    1. ****<u>Message transfer</u>****: A broadcast transfer that contains a serialized message.
    
    2. ****<u>Service transfer</u>****: A unicast transfer that contains either a service request or a service response.
  
  - Both message and service transfers can be further distinguished between:
    
    1. ****<u>Single-frame transfer</u>****: A transfer that is entirely contained in a single CAN frame.
    
    2. ****<u>Multi-frame transfer</u>****: A transfer that has its payload distributed over multiple CAN frames.
  
  - ****<u>Message Broadcasting</u>:**** A broadcast message is carried by a single message transfer that contains the serialized message.
    
    - A broadcast message has the following properties:
    
    | Property       | Description                                                                                                                                                          |
    | -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | Payload        | Serialized Message                                                                                                                                                   |
    | Data Type ID   | An identifier that indicates how the message should be interpreted                                                                                                   |
    | Source Node ID | Node ID of the node that has transmitted the transfer                                                                                                                |
    | Priority       | A positive integer value that defines the message urgency (0 is the highest priority). Higher priority transfers can delay transmission of lower priority transfers. |
    | Transfer ID    | An integer value that allows receiving nodes to distinguish this transfer from all others.                                                                           |
    
    - In order to broadcast a message, the broadcasting node must have a node ID that is unique within the network.
    
    - ****<u>Anonymous Message Broadcasting</u>:**** An anonymous message is a transfer that originates from a node that does not have a node ID.
      
      - A node that does not have a node ID is said to be in `passive mode`.
      
      - An anonymous message has the same properties as a regular message, except for source node ID.
      
      - An anonymous transfer can only be a single-frame transfer. Multi-frame anonymous messages are not allowed.
    
    - ****Timing:**** Message transmission should be aborted if it cannot be completed in 1 second.

#### <u>Payload Decomposition</u>:

- ****<u>Single Frame Transfer</u>:**** If the size of the entire transfer payload does not exceed the space available for payload in a single CAN frame, the whole transfer will be contained in one CAN frame.
  
  - If the size of the entire transfer payload does not exceed the space available for payload in a single CAN frame.

- ****<u>Multi Frame Transfer</u>:**** are used when the size of the transfer payload exceeds the space available for payload in a single CAN frame.
  
  - In order to make a `multi-frame transfer`, the node must first compute a CRC for the transfer payload.
  
  - The node prepends the CRC value to the transfer payload, and emits this data in chunks as a sequence of CAN frames (i.e. the first CAN frame contains the CRC and the first bytes of the payload).
  
  - All frames of a multi-frame transfer should be pushed to the bus at once, in the proper order from the first frame to the last frame.
  
  - ****Toggle Bit****: is a property defined at the CAN frame level. Its purpose is to detect and avoid CAN frame duplication errors.
    
    - The toggle bit of the first CAN frame of a multi-frame transfer must be set to zero.
    
    - The toggle bits of the following CAN frames of the transfer must alternate, i.e. the toggle bit of the second CAN frame will be one, the toggle bit of the third CAN frame will be zero, and so on.

### <u>Data Structure Description Language(DSDL)</u>:

- is used to define data structures for exchange via the CAN bus.

- The `DSDL definitions` are used to automatically generate the message `serialization/deserialization` code for a certain programming language.

- The tool that generates source code from DSDL definition files is called the `DSDL compiler`.

- ****<u>File Hierarchy</u>****:
  
  - Each DSDL definition file specifies exactly one data structure that can be used for message broadcasting.
  
  - The DSDL source file must be named using the `data type name` and `default data type ID`  as shown below:
    
    - ```bash
      [default data type ID.]<data type name>.uavcan
      ```
  
  - A defined data structure must be contained in a `namespace`, which may in turn to be `nested` within another namespace.
  
  - A namespace that is not nested in another namespace is called a `root namespace`.
  
  - For example, all standard data types are contained in the root namespace `uavcan`, which contains nested namespaces: `equipment`, `protocol`, etc.

```bash
+ uavcan                        <-- Root namespace
    + equipment                 <-- Nested namespace
        + ...
    + protocol                  <-- Nested namespace
        + 341.NodeStatus.uavcan <-- Definition of data type "uavcan.protocol.NodeStatus" with default data type ID 341
        + ...
    + Timestamp.uavcan          <-- Definition of data type "uavcan.Timestamp", default data type ID is not assigned
```

##### **<u>Syntax</u>:**

- A data structure definition consists of `attributes` and `directives`. Any line of the definition file may contain at most one attribute definition or at most one directive. The same line cannot contain an attribute definition and a directive at the same time.

- An `Attribute` can be either of the following:
  
  - ****Field****: A variable that can be modified by the application and exchanged via the network.
  
  - ****Constant****: An immutable value that does not participate in network exchange.

- A DSDL definition for a message data type may contain only the following:
  
  - Attribute definitions (zero or more)
  - Comments (optional)

****<u>Attribute Definition</u>:****

- Field definition patterns:   
  
  - ```bash
    cast_mode field_type field_name
    ```
  
  - ```bash
    cast_mode field_type[X] field_name
    ```
  
  - cast_mode field_type[<X] field_name
  
  - ```bash
    cast_mode field_type[<=X] field_name
    ```
  
  - ```bash
    void_type
    ```

- Constant definiton patterns:
  
  - ```bash
    cast_mode constant_type constant_name = constant_initializ
    ```

Discussion of Each Component:

- ****<u>Field Type</u>****: can be either a primitive data type or a nested data structure.
  
  - A `primitive data type` can be referred simply by name, e.g., `float16`, `bool`.
  
  - A field type name can be appended with a statement in square brackets to define an array:
    
    - Syntax `[X]` is used to define a static array of size exactly X items.
    - Syntax `[<X]` is used to define a dynamic array of size from 0 to X-1 items, inclusively.
    - Syntax `[<=X]` is used to define a dynamic array of size from 0 to X items, inclusively.
    - Arrays of maximum size with less than one item are not allowed. Multidimensional arrays are not allowed.

- ****<u>Field Name &amp; Constant Name</u>:**** For a message data type, all attributes must have a unique name within the data type.

- ****<u>Cast Mode</u>:**** defines the rules of conversion from the native value of a certain programming language to the serialized field value.
  
  - Cast mode may be left undefined, in which case the default will be used.
  
  - Cast modes are given below:
    
    - ****Saturated****: is the default cast mode, which will be used if the attribute definition does not specify the cast mode explicitly.
      
      - For `integers`, it prevents an integer overflow - for example, attempting to write 0x44 to a 4-bit field will result in a bitfield value of 0x0F.
      
      - For `floating point` values, it prevents overflow when casting to a lower precision floating point representation - for example, 65536.0 will be converted to a `float16` as 65504.0
    
    - ****Truncated:**** For integers, it discards the excess most significant bits - for example, attempting to write 0x44 to a 4-bit field will produce 0x04.
      
      - For floating point values, overflow during downcasting will produce an infinity.

- ****<u>Constant Definition</u>:****
  
  - A constant must be a primitive scalar type (i.e., arrays and nested data structures are not allowed as constant types).
  
  - A constant must be assigned with a constant initializer, which must be one of the following:
    
    - Integer zero (0).
    - Integer literal in base 10, starting with a non-zero character. E.g., 123, -12.
    - Integer literal in base 16 prefixed with `0x`. E.g., 0x123, -0x12, +0x123.
    - Integer literal in base 2 prefixed with `0b`. E.g., 0b1101, -0b101101, +0b101101.
    - Integer literal in base 8 prefixed with `0o`. E.g., 0o123, -0o777, +0o777.
    - Boolean `true` or `false`.

- ****<u>Void Type</u>:**** is a special field type that is intended for data alignment purposes. The specification defines 64 distinct void types as follows:
  
  - `void1` - 1 padding bit;
  
  - `void2` - 2 padding bits;
  
  - ....
  
  - `void63` - 63 padding bits;
  
  - `void64` - 64 padding bits;

---

##### Referrences:

- [[MAVLink Developer Guide | MAVLink Guide](https://mavlink.io/en/) & [MAVLink Basics — Dev documentation](https://ardupilot.org/dev/docs/mavlink-basics.html)]

- [Data structure description language - DroneCAN](https://dronecan.github.io/Specification/3._Data_structure_description_language/)

- [CAN bus transport layer - DroneCAN](https://dronecan.github.io/Specification/4.1_CAN_bus_transport_layer/)

- [Basic concepts - DroneCAN](https://dronecan.github.io/Specification/2._Basic_concepts/)

- [What is a gimbal? An easy-to-understand explanation of its mechanism, types, and how to use it for beginners - DJI Global or Other Regions](https://www.dji.com/global/media-center/insights/what-is-gimbal)

- [What is an IMU and what is it used for? | UAV Navigation](https://www.uavnavigation.com/products/ahrs-imu/what-is-an-imu)

- [Global Navigation Satellite System [Explained]](https://www.advancednavigation.com/tech-articles/global-navigation-satellite-system-gnss-and-satellite-navigation-explained/)

- [Electronic Speed Controllers (ESC): A Comprehensive Guide to Working, Features, Types and Applications.](https://mechtex.com/blog/a-comprehensive-guide-to-electronic-speed-controllers)

- [Multirotor - Wikipedia](https://en.wikipedia.org/wiki/Multirotor)

- [Drone Flight Controllers: A Comprehensive Guide | Grepow](https://www.grepow.com/blog/what-is-a-drone-flight-controller.html)

- [Drone Motors Explained: What They Are, Types & Components](https://mechtex.com/blog/basic-of-drone-motor-what-they-are-their-types-and-their-components)
