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

- 
