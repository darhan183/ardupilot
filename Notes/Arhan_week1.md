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

- 
