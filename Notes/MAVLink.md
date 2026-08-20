### <u>MAVLink Documentation</u>

- <b>What is MAVLink ?</b> [[MAVLink Developer Guide | MAVLink Guide](https://mavlink.io/en/) & [MAVLink Basics &mdash; Dev documentation](https://ardupilot.org/dev/docs/mavlink-basics.html)]
  
  - MAVLink = `Micro Air Vehicle Link`
  
  - MAVLink is a lightweight messaging protocol for communicating with drones.
  
  - NAVLink follows modern hybrid `publish-subscribe` & `point to point` design pattern.
    
    - Data Streams are sent/published as topics while configuration sub-protocols such as `mission-protocol` or `parameter protocol` are `point to point` with retransmission.
  
  - MAVLink messages can be sent over almost any serial connection.
  
  - Messages are defined within `XML files`.
    
    - Each `XML files` defines `message set` supported by particular MAVLink system also referred to as `Dialect`.
    
    - The reference message set  that is implemented by `Ground Control Stations` (GCS) & autopilot is defined in `common.xml`. 
  
  - <b><u>Message Format</u></b>:- 
    
    <img src="file:///home/d.arhan/Pictures/Screenshots/Screenshot%20from%202026-07-06%2017-25-46.png" title="" alt="Screenshot from 2026-07-06 17-25-46.png" width="724">
    
                                       FIG-1 MAVLink 1 Message Format

| Byte Index     | Content                       | Value         | Explanation                                                                                      |
|:--------------:|:-----------------------------:|:-------------:|:------------------------------------------------------------------------------------------------:|
| 0              | Packet Start Sign             | -             | Indicates the start of a new packet                                                              |
| 1              | Payload Length                | 0-255         | Indicates length of the following payload                                                        |
| 2              | Packet Sequence               | 0-255         | Each components counts up his send sequence. Allows to detect packet loss                        |
| 3              | System ID                     | 1-255         | ID of the SENDING system. Allows to differentiate different MAVs on the same network             |
| 4              | Component ID                  | 0-255         | ID of SENDING component. Allows to differentiate different components of same system             |
| 5              | Message ID                    | 0-255         | ID of the message - the ID defines what the payload "means" & how it should be correctly decoded |
| 6 to (n+6)     | Data                          | (0-255) bytes | Data of the message, depends on the message ID.                                                  |
| (n+7) to (n+8) | Checksum(Low byte, high byte) | -             | -                                                                                                |

#### <u>MAVLink(v1) vs MAVLink(v2)</u>:-

| Features               | MAVLink 1     | MAVLink 2                              |
| ---------------------- | ------------- | -------------------------------------- |
| Header Size            | 6 bytes       | 10 bytes                               |
| Message ID             | 1 byte(0-255) | 3 bytes(0-16,777,215)                  |
| Packet Signing         | Not Supported | 13-byte signature                      |
| Extension Field        | No            | Yes                                    |
| Payload Truncation     | No            | Removes trailing zero bytes            |
| Backward Compatibility | -             | Can communicate with MAVLink 1 devices |

---

#### <u>Why was MAVLink Created ?</u>

- <b>MAVLink</b> was created in 2009 by Lorenz Meier at ETH Zurich to provide a lightweight, reliable communication protocol for unmanned systems like drones.

- It was designed to transmit real-time telemetry and commands efficiently over low-bandwidth, high-latency channels between autonomous vehicles and Ground Control Stations (GCS). 

- Before MAVLink, there was no standardized way for different autopilots and ground stations to talk to each other. It was developed to solve several critical challenges in the early days of autonomous robotics.

- Drones communicate over long-distance, low-bandwidth radio waves. MAVLink was built as a ultra-lightweight, binary protocol to fit critical data into tiny message packets.

- Radio connections to drones frequently drop or suffer from interference. MAVLink was engineered to be "lossy," meaning the system can lose packets without crashing the entire communication stream.

- The protocol introduced standardized error-checking (checksums). If a radio glitch corrupts a command packet, the drone detects it instantly and ignores the bad data to prevent crashes.

---

### <u>MAVLink Versions</u>

- Currently `MAVLink Version = MAVLink 2.0`, also supports `MAVLink 1.0`.

- <b><u>Determining Protocol/Message Version</u>:</b> 
  
  - The major version can be determined from the packet start marker byte:
    
    - MAVLink 1: `0xFE`
    - MAVLink 2: `0xFD`

---

### <u>MAVLink 2</u>

- The new features of `MAVLink 2` are:
  
  - **24 bit message ID**: Allows over 16 million unique message definitions in a dialect (MAVLink 1 was limited to 256).
  
  - <b>Packet Signing</b> : Authenticate that messages were sent by trusted systems.
  
  - **Message Extensions:** Add new fields to existing MAVLink message definitions without breaking binary compatibility for receivers that have not updated.
  
  - **Empty-byte Payload Truncation:** Empty (zero-filled) bytes at the end of the serialized payload must be removed before sending.
  
  - **Compatibility/Incompatibility Flags:** Packets with compatibility flags can still be handled in the standard way, while packets with incompatibility flags must be dropped if the flag is not supported.

- <b><u>Message/Packet Signing</u>:</b> 
  
  - <b>Frame Format</b>:  For a signed packet the **0x01** bit of the `incompatibility flag field` is set true and an additional 13 bytes of `signature` data appended to the packet.
    
    ![](/home/d.arhan/Pictures/Screenshots/2026-08-20-21-18-14-image.png)
    
        Fig2: MAVLink 2 Frame Format
  
  - The 13 bytes of signature are:
  
  | Data               | Description                                                                                 |
  | ------------------ | ------------------------------------------------------------------------------------------- |
  | linkID(8 Bits)     | ID of link on which packet is sent.                                                         |
  | timestamp(48 Bits) | This must monotonically increase for every message on a particular link.                    |
  | signature(48 Bits) | A 48 bit signature for the packet, based on the complete packet, timestamp, and secret key. |
  
  - <u><b>Link IDs:</b></u>  The 8 bit link ID is provided to ensure that the signature system is robust for multi-link MAVLink systems.
    
    - Each implementation should assign a `link ID` to each of the MAVLink communication channels it has enabled and should put this ID in the link wID field.
    
    - The monotonically increasing timestamp rule is applied separately for each logical stream, where a stream is defined by the tuple:
      
      ```bash
      (SystemID,ComponentID,LinkID)
      ```
  
  - <b><u>Signature</u>:</b> 
