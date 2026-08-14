### <u>DroneCAN</u>:

#### <u>Basic Concepts:</u> [Basic concepts - DroneCAN](https://dronecan.github.io/Specification/2._Basic_concepts/)

- is a lightweight protocol designed to provide a highly reliable communication method for robotic application via `CAN BUS`.

- The DroneCAN network is a decentralized peer network, where each peer (node) has a unique numeric identifier - `node ID`.

- The nodes of DroneCAN network can communicate using any of the following communication methods:
  
  - <b><u>Message Broadcasting</u></b>: The primary method of data exchange with publish/subscribe methods.
  
  - <b><u>Service Invocation</u></b>: The communication method for peer-to-peer request/response interactions.  

- A predefined set of data structures is used, where each data structure has a unique identifier - the `Data Type ID (DTID)`.

- Some data structures are standard and defined by the protocol specification; others may be specific to a particular application.

- A pair of `data type ID` and `node ID` can be used to support redundant nodes with identical functionality inside the same network.

- Message and service data structures are defined using the `Data Structure Description Language (DSDL)`.
  
  - The DSDL description is used to generate the serialization/deserialization code for a given data structure in each target programming language.
  
  - <b>Serialization:</b> is the process of translating a data structure or object state into storable or transmissible sequence of bytes or strings.
  
  - <b>DeSerialization:</b> is the process of extracting a data structure from series of bytes.
    
    <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-08-06-20-05-36-image.png" title="" alt="" width="634">
    
        Fig1: Serialization & DeSerialization Flow Diagram

- Serialized message and service data structures are exchanged by means of the CAN bus transport layer, which implements automatic decomposition of long transfers into several CAN frames, allowing nodes to exchange data structures of arbitrary size.

                                                                                           <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-08-06-22-10-05-image.png" title="" alt="" width="257">

    Fig2: DroneCAN Architecture

---

#### <u>Message Broadcasting</u>:

- It refers to the  transmission of a serialized data structure over CAN bus to other nodes.

- This is the primary DroneCAN data exchange mechanism.

- Typical use cases may include transfer of the following kinds of data: **sensor measurements, actuator commands, or equipment status information.**

- A broadcast message includes the following information:
  
  | Field          | Content                                                                                                   |
  |:--------------:|:---------------------------------------------------------------------------------------------------------:|
  | Payload        | The serialized data structure                                                                             |
  | Data type ID   | Numerical identifier that indicates how the data structure should be interpreted                          |
  | Source node ID | The node ID of the transmitting node                                                                      |
  | Transfer ID    | A small overflowing integer that increments with every transfer of this type of message from a given node |

- ##### Anonymous message broadcasting:
  
  - Nodes that don’t have a unique node ID can publish anonymous messages.
  
  - An anonymous message is different from a regular message in that it doesn’t contain source node ID.
  
  - This kind of data exchange is useful during initial configuration of the node, particularly during dynamic node ID allocation procedure.

#### <u>Service Invocation</u>:

- Service invocation is a two-step data exchange between two nodes: `A Client and A Server`. The steps are given below:
  
  1. The client sends a service request to the server.
  
  2. The server takes appropriate actions and sends a response to the client.

- Use cases for this type of communication include: <b>node configuration parameter update, firmware update, file transfer</b>.
  
  | Field          | Content                                                                                       |
  |:--------------:|:---------------------------------------------------------------------------------------------:|
  | Payload        | The serialized data structure                                                                 |
  | Data type ID   | Numerical identifier that indicates how the data structure should be interpreted              |
  | Client node ID | Source node ID during request transfer, destination node ID during response transfer          |
  | Server node ID | Destination node ID during request transfer, source node ID during response transfer          |
  | Transfer ID    | A small overflowing integer that increments with every call to this service from a given node |

- Both `request and response` contain exactly the same values for all fields except payload, where the content is application defined. Clients can match the response with a corresponding request using the following fields: `data type ID, client node ID, server node ID, and transfer ID`.

---

#### <u>CAN Bus Transport Layer</u>: [CAN bus transport layer - DroneCAN](https://dronecan.github.io/Specification/4.1_CAN_bus_transport_layer/)

- <b>The Concept of Transfer:</b> A Transfer is an act of data transmission between nodes.
  
  - A transfer that is addressed to one particular node is a `unicast transfer`.
  
  - A transfer that is addressed to all nodes except the source node is a `broadcast transfer`.
  
  - DroneCAN defines the following types of transfers:
    
    1. <b><u>Message transfer</u></b>: A broadcast transfer that contains a serialized message.
    
    2. <b><u>Service transfer</u></b>: A unicast transfer that contains either a service request or a service response.
  
  - Both message and service transfers can be further distinguished between:
    
    1. <b><u>Single-frame transfer</u></b>: A transfer that is entirely contained in a single CAN frame.
    
    2. <b><u>Multi-frame transfer</u></b>: A transfer that has its payload distributed over multiple CAN frames.
  
  - <b><u>Message Broadcasting</u>:</b> A broadcast message is carried by a single message transfer that contains the serialized message.
    
    - A broadcast message has the following properties:
    
    | Property       | Description                                                                                                                                                          |
    |:--------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
    | Payload        | Serialized Message                                                                                                                                                   |
    | Data Type ID   | An identifier that indicates how the message should be interpreted                                                                                                   |
    | Source Node ID | Node ID of the node that has transmitted the transfer                                                                                                                |
    | Priority       | A positive integer value that defines the message urgency (0 is the highest priority). Higher priority transfers can delay transmission of lower priority transfers. |
    | Transfer ID    | An integer value that allows receiving nodes to distinguish this transfer from all others.                                                                           |
    
    - In order to broadcast a message, the broadcasting node must have a node ID that is unique within the network.
    
    - <b><u>Anonymous Message Broadcasting</u>:</b> An anonymous message is a transfer that originates from a node that does not have a node ID.
      
      - A node that does not have a node ID is said to be in `passive mode`.
      
      - An anonymous message has the same properties as a regular message, except for source node ID.
      
      - An anonymous transfer can only be a single-frame transfer. Multi-frame anonymous messages are not allowed.
    
    - ****Timing:**** Message transmission should be aborted if it cannot be completed in 1 second.
  
  - <b><u>Service Invocation</u>:</b> It consists of two service transfers:
    
    1. <b>Service Request Transfer</b>: from the node that invokes the service, known as `client`, to the node that provides the service, known as `server`.
    
    2. <b>Service Response Transfer</b>: once the `server node` receives the `service request`and processes it, it sends a `response transfer` back to the `client`.
    - A `service request transfer` has the following properties:
      
      | Property            | Description                                                                                                                                                                           |
      |:-------------------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
      | Payload             | The serialized request structure                                                                                                                                                      |
      | Data type ID        | An identifier that indicates how the request should be interpreted                                                                                                                    |
      | Source node ID      | Node ID of the client                                                                                                                                                                 |
      | Destination node ID | Node ID of the server                                                                                                                                                                 |
      | Priority            | A positive integer value that defines the message urgency (0 is the highest priority). Higher priority transfers can delay transmission of lower priority transfers.                  |
      | Transfer ID         | An integer value that:<br/> 1. allows the server to distinguish this request from other requests from the same client<br/>2. allows the client to match the response with its request |
      
      - A `service response transfer` has the following properties:
        
        | Property            | Description                                         |
        |:-------------------:|:---------------------------------------------------:|
        | Payload             | The serialized response structure                   |
        | Data type ID        | Must be the same value as in the request transfer   |
        | Source node ID      | Node ID of the server                               |
        | Destination node ID | Node ID of the client                               |
        | Priority            | Should be the same value as in the request transfer |
        | Transfer ID         | Must be the same value as in the request transfer   |
    
    - Both client and server must have node ID values that are unique within the network.
    
    - <b>Timing</b>: The following timings should be used,
      
      - Service transfer transmission should be aborted if does not complete in 1 second.
      
      - The client should stop waiting for a response from the server if one has not arrived within 1 second.
      
      - The server should be able to process any request in under 0.5 seconds.
      
      - If the different values are used, they must be documented

---

#### <u>Transmission</u>:

- <b><u>Transfer ID Computation</u>:</b> The `Transfer ID` is a small unsigned integer value that is added to every outgoing transfer, it is used by `receiving nodes` to distinguish between individual transfers.
  
  - The logic to compute the `Transfer ID` relies on the concept of `Transfer Descriptor`.
  
  - A transfer descriptor is a set of properties that identify a particular set of transfers that originate from the `same node`, share the same `data type ID` and the same `transfer type`.
  
  - The properties that constitute a transfer descriptor are listed below:
    
    - Transfer type (message broadcast, service request, etc.)
    
    - Data type ID
    
    - Source Node ID
    
    - Destination Node ID (only for unicast transfers)
  
  - Every node that needs to publish a transfer must maintain the mapping from `transfer descriptors` to `transfer ID`. This mapping is referred to as the `transfer ID map`.

---

#### <u>Payload Decomposition</u>:

- <b><u>Single Frame Transfer</u>:</b> If the size of the entire transfer payload does not exceed the space available for payload in a single CAN frame, the whole transfer will be contained in one CAN frame.
  
  - If the size of the entire transfer payload does not exceed the space available for payload in a single CAN frame.

- <b><u>Multi Frame Transfer</u>:</b> are used when the size of the transfer payload exceeds the space available for payload in a single CAN frame.
  
  - In order to make a `multi-frame transfer`, the node must first compute a CRC for the transfer payload.
  
  - The node prepends the CRC value to the transfer payload, and emits this data in chunks as a sequence of CAN frames (i.e. the first CAN frame contains the CRC and the first bytes of the payload). 
  
  - All frames of a multi-frame transfer should be pushed to the bus at once, in the proper order from the first frame to the last frame.
  
  - <b>Toggle Bit</b>: is a property defined at the CAN frame level. Its purpose is to detect and avoid CAN frame duplication errors.
    
    - The toggle bit of the first CAN frame of a multi-frame transfer must be set to zero.
    
    - The toggle bits of the following CAN frames of the transfer must alternate, i.e. the toggle bit of the second CAN frame will be one, the toggle bit of the third CAN frame will be zero, and so on.

---

#### <u>CAN Frame Format</u>:

- `DroneCAN` uses only `CAN 2.0B` frame format (29-bit identifiers). DroneCAN can share the same bus with other protocols based on `CAN 2.0A` (11-bit identifiers).

- <b><u>ID Field</u></b>: 
  
  1. <b><u>Message Frame</u>:</b> Contents of the CAN ID field depend on the transfer type.
     
     <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-08-13-12-51-51-image.png" title="" alt="" width="645">
     
     | Field               | Bits | Allowed Values | Description                         |
     |:-------------------:|:----:|:--------------:|:-----------------------------------:|
     | Priority            | 5    | Any            |                                     |
     | Message Type ID     | 16   | Any            | Data type ID of the encoded message |
     | Service not message | 1    | 0              | Always Zero                         |
     | Source Node ID      | 7    | 1-127          |                                     |
  
  2- <b><u>Anonymous Message Frame</u>:</b> 
  
  <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-08-13-13-13-48-image.png" title="" alt="" width="718">
  
  | Field                         | Bits | Allowed Values | Description                         |
  |:-----------------------------:|:----:|:--------------:|:-----------------------------------:|
  | Priority                      | 5    | Any            |                                     |
  | Discriminator                 | 14   | Any            |                                     |
  | Lower bits of message type ID | 2    | Any            | Data type ID of the encoded message |
  | Service not message           | 1    | 0              | Always zero                         |
  | Source node ID                | 7    | 0              | Always zero                         |
  
  3- <b><u>Service Message Frame</u>:</b>
  
     <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-08-13-13-23-55-image.png" title="" alt="" width="725"> 
  
  | Field                | Bits | Allowed Values | Description                                                         |
  |:--------------------:|:----:|:--------------:|:-------------------------------------------------------------------:|
  | Priority             | 5    | Any            |                                                                     |
  | Service type ID      | 8    | Any            | Data type ID of the encoded service request or response             |
  | Request not response | 1    | Any            | Values: 1 - service request transfer, 0 - service response transfer |
  | Destination node ID  | 7    | 1-127          |                                                                     |
  | Service not message  | 1    | 1              | Always one                                                          |
  | Source node ID       | 7    | 1-127          |                                                                     |
  
  - <b>Priority</b>: Valid values for priority range from 0 to 31, inclusively, where 0 corresponds to highest priority (and 31 corresponds to lowest priority).
    
    - In `multi-frame transfers`, the value of the `priority` field must be identical for all frames of the transfer.
  
  - <b>Message Type ID</b>: Valid values of message type ID range from 0 to 65535, inclusively.
    
    - Valid values of message type ID range for `anonymous message transfers` range from 0 to 3, inclusively. This limitation is due to the fact that only 2 lower bits of the message type ID are available in this case.
  
  - <b>Service Type</b>: Valid values of service type ID range from 0 to 255, inclusively.
  
  - <b>Node ID</b>: Valid values of Node ID range from 1 to 127, inclusively.

---

#### <u>Payload</u>:

<img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-08-13-14-13-42-image.png" title="" alt="" width="478">

| Field            | Description                                                                                |
|:----------------:|:------------------------------------------------------------------------------------------:|
| Transfer Payload | Actual payload of the transfer                                                             |
| Tail byte        | The last byte of the CAN frame data field, which contains auxiliary transport layer fields |

- The tail byte contains the following fields, starting from the most significant bit:
  
  | Field             | Bits |
  |:-----------------:|:----:|
  | Start of transfer | 1    |
  | End of transfer   | 1    |
  | Toggle bit        | 1    |
  | Transfer ID       | 5    |

- <b>Single Frame Transfer:</b> 
  
  <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-08-13-19-47-56-image.png" title="" alt="" width="714">
  
                              Data Frame of Single CAN frame

- <b>Multi Frame Transfer:</b> 
  
  <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-08-13-19-49-17-image.png" title="" alt="" width="720">
  
                            Data Frame of First CAN frame
  
  <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-08-13-19-49-46-image.png" title="" alt="" width="719">
  
                 Data Field of Following CAN frames except the last one
  
  <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-08-13-19-50-02-image.png" title="" alt="" width="722">
  
                                Data Field of last CAN Frame
  
  - The tail byte contains the following fields,
    
    - ****Start of Transfer(SOF)****: For single-frame transfers, `Start of Transfer (SOF) = 1`.
      
      - For multi-frame transfers, `Start of Transfer (SOF) = 1` if current frame is `first frame` of transfer & `SOF = 0` otherwise.
    
    - ****End of Transfer(EOF):**** For single-frame transfers, `EOF = 1`.
      
      - For multi-frame transfers, `EOF = 1` if the current frame is the `last frame` of the transfer & `EOF = 0` otherwise.
    
    - ****Toggle Bit:**** For single-frame transfers, `Toggle Bit = 0`.
      
      - For `multi-frame transfers`, this field contains the value of the toggle bit. This will alternate value between frames, starting at `0` for the first frame.
    
    - <b>Transfer ID:</b> This field contains the transfer ID value of the current transfer for all types of transfers.
      
      - The value is 5 bits wide, therefore the allowed values range from 0 to 31, inclusively.

---

### <u>Data Structure Description Language(DSDL)</u>: [Data structure description language - DroneCAN](https://dronecan.github.io/Specification/3._Data_structure_description_language/)

- is used to define data structures for exchange via the CAN bus.

- The `DSDL definitions` are used to automatically generate the message `serialization/deserialization` code for a certain programming language.

- The tool that generates source code from DSDL definition files is called the `DSDL compiler`.

- <b> <u>File Hierarchy</u></b>:
  
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

---

##### <b><u>Syntax</u>:</b>

- A data structure definition consists of `attributes` and `directives`. Any line of the definition file may contain at most one attribute definition or at most one directive. The same line cannot contain an attribute definition and a directive at the same time.

- An `Attribute` can be either of the following:
  
  - <b>Field</b>: A variable that can be modified by the application and exchanged via the network.
  
  - <b>Constant</b>: An immutable value that does not participate in network exchange.

- A `Directive` is a statement that provides instructions to the DSDL compiler.

- A DSDL definition for a message data type may contain only the following:
  
  - Attribute definitions (zero or more)
  - Directives (zero or more)
  - Comments (optional)

<b><u>Attribute Definition</u>:</b> 

- Field definition patterns:   
  
  - ```bash
    cast_mode field_type field_name
    ```
  
  - ```bash
    cast_mode field_type[X] field_name
    ```
  
  -     cast_mode field_type[<X] field_name
  
  - ```bash
    cast_mode field_type[<=X] field_name
    ```
  
  - ```bash
    void_type
    ```

- Constant definiton patterns:
  
  - ```bash
    cast_mode constant_type constant_name = constant_initializer
    ```

- Discussion of Each Component:
  
  - <b><u>Field Type</u></b>: can be either a primitive data type or a nested data structure.
    
    - A `primitive data type` can be referred simply by name, e.g., `float16`, `bool`.
    
    - A field type name can be appended with a statement in square brackets to define an array:
      
      - Syntax `[X]` is used to define a static array of size exactly X items.
      - Syntax `[<X]` is used to define a dynamic array of size from 0 to X-1 items, inclusively.
      - Syntax `[<=X]` is used to define a dynamic array of size from 0 to X items, inclusively.
      - Arrays of maximum size with less than one item are not allowed. Multidimensional arrays are not allowed.
  
  - <b><u>Field Name & Constant Name</u>:</b> For a message data type, all attributes must have a unique name within the data type.
  
  - <b><u>Cast Mode</u>:</b> defines the rules of conversion from the native value of a certain programming language to the serialized field value.
    
    - Cast mode may be left undefined, in which case the default will be used.
    
    - Cast modes are given below:
      
      - <b>Saturated</b>: is the default cast mode, which will be used if the attribute definition does not specify the cast mode explicitly.
        
        - For `integers`, it prevents an integer overflow - for example, attempting to write 0x44 to a 4-bit field will result in a bitfield value of 0x0F.
        
        - For `floating point` values, it prevents overflow when casting to a lower precision floating point representation - for example, 65536.0 will be converted to a `float16` as 65504.0
      
      - <b>Truncated:</b> For integers, it discards the excess most significant bits - for example, attempting to write 0x44 to a 4-bit field will produce 0x04.
        
        - For floating point values, overflow during downcasting will produce an infinity.
  
  - <b><u>Constant Definition</u>:</b> 
    
    - A constant must be a primitive scalar type (i.e., arrays and nested data structures are not allowed as constant types).
    
    - A constant must be assigned with a constant initializer, which must be one of the following:
      
      - Integer zero (0).
      - Integer literal in base 10, starting with a non-zero character. E.g., 123, -12.
      - Integer literal in base 16 prefixed with `0x`. E.g., 0x123, -0x12, +0x123.
      - Integer literal in base 2 prefixed with `0b`. E.g., 0b1101, -0b101101, +0b101101.
      - Integer literal in base 8 prefixed with `0o`. E.g., 0o123, -0o777, +0o777.
      - Boolean `true` or `false`.
  
  - <b><u>Void Type</u>:</b> is a special field type that is intended for data alignment purposes. The specification defines 64 distinct void types as follows:
    
    - `void1` - 1 padding bit;
    
    - `void2` - 2 padding bits;
    
    - ....
    
    - `void63` - 63 padding bits;
    
    - `void64` - 64 padding bits;
    
    - A field of type void does not have a name and its cast mode cannot be specified.During message serialization, all void fields must be populated with zero bits; during deserialization, contents of the void fields should be ignored.

---

#### <u>Primitive Data Types</u>

| Name    | Bit Length | Possible representation in C/C++      | Value range           | Binary representation                                         |
|:-------:|:----------:|:-------------------------------------:|:---------------------:|:-------------------------------------------------------------:|
| bool    | 1          | bool                                  | {0,1}                 | One Bit                                                       |
| intX    | 2 ≤ X ≤ 64 | int8_t, int16_t, int32_t, int64_t     | [-(2^X)/2, 2^X/2 - 1] | Two's Complement                                              |
| uintX   | 2 ≤ X ≤ 64 | uint8_t, uint16_t, uint32_t, uint64_t | [0, 2^X - 1]          |                                                               |
| float16 | 16         | float                                 | ±65504                |                                                               |
| float32 | 32         | float                                 | Approx. ±1039         |                                                               |
| float64 | 64         | double                                | Approx. ±10308        |                                                               |
| voidX   | 1 ≤ X ≤ 64 |                                       |                       | X zero bits (set to zero when encoding, ignore when decoding) |

----

#### <u>Naming Rules</u>

- Field names, constant names, and type names must contain only `ASCII alphanumeric characters` and `underscores ([A-Za-z0-9_]`), and must begin with an `ASCII alphabetic character ([A-Za-z]`).

- Violation of this rule must be detected by the DSDL compiler and treated as a fatal error.

- <b><u>Optional</u></b>:
  
  - Field and namespace names should be all-lowercase words separated with underscores, and may include numbers, (e.g.: `field_name`, `my_namespace_7`).
  
  - Constant names should be all-uppercase words separated with underscores, and may include numbers (e.g.: `CONSTANT_NAME`).
  
  - Data type names should be in `camel case` and may include numbers (e.g.: `TypeName`, `TypeName2`).

---

#### <u>Data Type Compatibility</u>

- <b>The Concept of Data Type Compatibility:</b> It is vital that all nodes exchanging some particular data structure use compatible DSDL definitions of it.
  
  - <b><u>Binary Layout</u>:</b> This implies that compatible data structures must have the `same field types in the same order`, as shown in the following example,
    
    - First definition:
      
      ```
      uint8 a
      uint8 b
      ```
    
    - Second definition:
      
      ```
      uint12 a
      uint4 b
      ```
    
    - Even though the bit length of the data structures above is the same, the binary layout is clearly not compatible.
    
    - **Example for compatible data type,**
    
    - First definition:
      
      ```
      uint8 a
      uint8 b
      ```
    
    - Second definition:
      
      ```
      uint8 a
      uint8 b
      ```
    
    - Both definitions have the same field types in the same order. Therefore, they have the same binary layout and can be interpreted correctly by both nodes.
  
  - <b><u>Field Name & Order</u></b>: This implies that compatible data structures must have the `same field types and names in the same order`.
    
    - First definition:
      
      ```
      uint8 a
      uint8 b
      ```
    
    - Second definition:
      
      
      ```
      uint8 b
      uint8 a
      ```
    
    - Even though the first and the second definitions share the same binary layout (two fields of type `uint8`), `they feature different field names and therefore are semantically incompatible`.

---
