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
  
  - <b><u>Signature</u>:</b> The 48 bit (6 byte) signature is the first 48 bits of a `SHA-256` hash of the complete packet (without the signature, but including the timestamp) appended to the `secret key`.
    
    - The secret key is 32 bytes of binary data stored on both ends of a MAVLink channel (i.e. an autopilot and a ground station).
    
    - ```bash
      signature = sha256_48(secret_key + header + payload + CRC + link-ID + timestamp)
      ```
      
      where `+` represents concatenation and `sha256_48()` is a sha256 implementation which returns the first 48 bits of the normal sha256 output.
  
  - <b><u>Timestamp</u>:</b> 
    
    - The timestamp is a 48 bit number with units of 10 microseconds.
    
    - All timestamps generated must be at least 1 more than the previous timestamp sent in the same session for the same link. The timestamp may get ahead of GMT time if there is a burst of packets at a rate of more than 100 thousand packets per second.

---

#### <u>Routing</u>:

- Each system has a network-unique `system ID`, and each component has a system-unique `component ID` that can be used for addressing/routing. Both are values between 1 and 255.

-  If the IDs are omitted or set to zero then the message is considered a `broadcast` (intended for all systems/components).

- `target_system`: System that should execute the command

- `target_component`: Component that should execute the command (requires `target_system`).

- <b><u>ID Allocation</u>:</b>  
  
  - MAVLink does not provide mechanisms to automate allocation of system and component IDs. Instead these are manually allocated by the system integrator.
  
  - Autopilots on a MAVLink network are allocated sequentially *increasing system IDs* from `1` (1 is usually the default autopilot system ID).
  
  - GCS and MAVLink developer APIs are allocated sequentially *decreasing system IDs*, starting from `255`.

- <b><u>Routing Detail</u>:</b> 
  
  - A component should attempt to process a message locally if any of these conditions hold:
    
    - The `target_system` field is omitted or has value `0` (network broadcast).
    
    - The `target_system` matches its system ID and the `target_component` field is omitted or has value `0` (system broadcast).
    
    - The `target_system` and `target_component` matches its system and component IDs.

---

#### <u>Packet Serialization</u>

- <b><u>MAVLink 2 Packet Format</u>:</b> 

![](/home/d.arhan/Pictures/Screenshots/2026-08-21-21-35-34-image.png)

| Byte Index                                                  | C version                | Content                              | Value        | Explanation                                                                                                                                                             |
|:-----------------------------------------------------------:|:------------------------:|:------------------------------------:|:------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| 0                                                           | uint8_t magic            | Packet start marker                  | 0xFD         | Protocol-specific start-of-text (STX) marker used to indicate the beginning of a new packet. Any system that does not understand protocol version will skip the packet. |
| 1                                                           | uint8_t len              | Payload length                       | 0 - 255      | Indicates length of the following payload section. This may be affected by payload truncation.                                                                          |
| 2                                                           | uint8_t incompat_flags   | Incompatibility Flags                |              | Flags that must be understood for MAVLink compatibility (implementation discards packet if it does not understand flag).                                                |
| 3                                                           | uint8_t compat_flags     | Compatibility Flags                  |              | Flags that can be ignored if not understood (implementation can still handle packet even if it does not understand flag).                                               |
| 4                                                           | uint8_t seq              | Packet sequence number               | 0 - 255      | Used to detect packet loss. Components increment value for each message sent.                                                                                           |
| 5                                                           | uint8_t sysid            | System ID (sender)                   | 1 - 255      | ID of system (vehicle) sending the message. Used to differentiate systems on network.                                                                                   |
| 6                                                           | uint8_t compid           | Component ID (sender)                | 1 - 255      | ID of component sending the message. Used to differentiate components in a system (e.g. autopilot and a camera).                                                        |
| 7 to 9                                                      | uint32_t msgid:24        | Message ID (low, middle, high bytes) | 0 - 16777215 | ID of message type in payload. Used to decode data back into message object.                                                                                            |
| For n-byte payload:<br/>n=0: NA, n=1: 10, n>=2: 10 to (9+n) | uint8_t payload[max 255] | Payload                              |              | Message data. Depends on message type (i.e. Message ID) and contents.                                                                                                   |
| (n+10) to (n+11)                                            | uint16_t checksum        | Checksum(low byte, high byte))       |              |                                                                                                                                                                         |
| (n+12) to (n+24)                                            | uint8_t signature[13]    | Signature                            |              | Signature to ensure the link is tamper-proof.                                                                                                                           |

- The minimum packet length is 12 bytes for acknowledgment packets without payload.

- The maximum packet length is 280 bytes for a signed message that uses the whole payload.

- <b><u>MAVLink 1 Packet Format</u>:</b> 
  
  <img src="file:///home/d.arhan/Pictures/Screenshots/2026-08-24-13-29-44-image.png" title="" alt="" width="709">
  
  | Byte Index                                                | C version                | Content                        | Value   | Explanation                                                                                                                                                             |
  | --------------------------------------------------------- | ------------------------ | ------------------------------ | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | 0                                                         | uint8_t magic            | Packet start marker            | 0xFE    | Protocol-specific start-of-text (STX) marker used to indicate the beginning of a new packet. Any system that does not understand protocol version will skip the packet. |
  | 1                                                         | uint8_t len              | Payload length                 | 0 - 255 | Indicates length of the following payload section (fixed for a particular message).                                                                                     |
  | 2                                                         | uint8_t seq              | Packet sequence number         | 0 - 255 | Used to detect packet loss. Components increment value for each message sent.                                                                                           |
  | 3                                                         | uint8_t sysid            | System ID                      | 1 - 255 | ID of system (vehicle) sending the message. Used to differentiate systems on network.                                                                                   |
  | 4                                                         | uint8_t compid           | Component ID                   | 1 - 255 | ID of component sending the message. Used to differentiate components in a system (e.g. autopilot and a camera).                                                        |
  | 5                                                         | uint8_t msgid            | Message ID                     | 0 - 255 | ID of message type in payload. Used to decode data back into message object.                                                                                            |
  | For n-byte payload:<br/>n=0: NA, n=1: 6, n>=2: 6 to (5+n) | uint8_t payload[max 255] | Payload data                   |         | Message data. Content depends on message type (i.e. Message ID).                                                                                                        |
  | (n+6) to (n+7)                                            | uint16_t checksum        | Checksum (low byte, high byte) |         |                                                                                                                                                                         |
  
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
    - `uint32_t`, `float`   - 4 bytes
    - `uint16_t`                  - 2 bytes
    - `uint8_t`, `char`       - 1 bytes
  
  - If two fields have the same length, their order is preserved as it was present before the data field size ordering.
  
  - Arrays are handled based on the data type they use, not based on the total array size.

- <b><u>Empty-Byte Payload Truncation(MAVLink 2)</u>:</b> `MAVLink 2` truncate any empty (zero-filled) bytes at the end of the serialized payload before it is sent. This contrasts with `MAVLink 1`, where bytes were sent for all fields regardless of content.

---

#### <u>MAVLink XML File Schema/Format</u>:

```bash
<?xml version="1.0"?>
<mavlink>

    <include>common.xml</include>
    <include>other_dialect.xml</include>

    <!-- NOTE: If the included file already contains a version tag, remove the version tag here, else uncomment to enable. -->
    <!-- <version>6</version> -->

    <dialect>8</dialect>

    <enums>
        <!-- Enums are defined here (optional) -->
    </enums>

    <messages>
        <!-- Messages are defined here (optional) -->
    </messages>

</mavlink>
```

- <b>include</b>: This tag is used to specify any other XML files included in your dialect.
  
  - can include multiple files using separate tags.
  
  - Nested `include` of files are supported (mavgen permits up to 5 levels of nesting).

- **enums:** Dialect-specific enums can be defined in this block (if none are defined in the file, the block is optional/can be removed).

- **messages:** Dialect-specific messages can be defined in this block (if none are defined in the file, the block is optional/can be removed).

- **<u>Enum Definitions(enums)</u>**: 
  
  - Enums are defined within the `<enums> ... </enums>` tags.
  
  - Enum values are defined within the enum using `<entry> ... </entry>` tags. For example,
    
    ```bash
    <enum name="LANDING_TARGET_TYPE">
        <description>Type of landing target</description>
        <entry value="0" name="LANDING_TARGET_TYPE_LIGHT_BEACON">
            <description>Landing target signaled by light beacon (ex: IR-LOCK)</description>
        </entry>
        <entry value="1" name="LANDING_TARGET_TYPE_RADIO_BEACON">
            <description>Landing target signaled by radio beacon (ex: ILS, NDB)</description>
        </entry>
        <entry value="2" name="LANDING_TARGET_TYPE_VISION_FIDUCIAL">
            <description>Landing target represented by a fiducial marker (ex: ARTag)</description>
        </entry>
        <entry value="3" name="LANDING_TARGET_TYPE_VISION_OTHER">
            <description>Landing target represented by a pre-defined visual shape/feature (ex: X-marker, H-marker, square)</description>
        </entry>
    </enum>
    ```
  
  - **<enums> Element**: Grouping element for `<enum>` elements.
  
  - **<enum> Element:** Defines a group of related values that can be set as the allowed values for a message `<field>`or command `<param>`. The values are defined in nested `<entry>` elements.
    
    - Attributes:
      
      - <b>name</b> (required): The name of the enum. **This is a string of capitalized, underscore-separated words**.
      
      - **bitmask** (optional):  Set to `true` for enums that define entries with values that increase by a power of 2, such as flags.
    
    - Nested elements:
      
      - **<description>** (optional): A string describing the purpose of the enum.
      
      - **<entry>** (optional): An entry (zero or more entries can be specified for each enum).
      
      - **Lifecycle elements** (optional): <wip>|<superseded>|<deprecated>.
  
  - **<entry> Element:** A named value within a particular `<enum>` element.
    
    - Attributes:
      
      - **name** (required): The name of the enum value. This is a string of capitalized, underscore-separated words.
      
      - **value** (optional): The value for the entry (a number).
    
    - Nested elements:
      
      - **<description>** (optional): A description of the entry.
      
      - **Lifecycle elements** (optional): <wip>|<superseded>|<deprecated>.

- **<u>Message Definition</u>:** Messages are defined within `<messages>...</messages>` tags.
  
  - Individual fields to be encoded in the message payload are defined using `<field> ... </field>` tags. Every message must have at least one field. For example,
    
    ```bash
    <message id="147" name="BATTERY_STATUS">
      <description>Battery information. Updates GCS with flight controller battery status. Smart batteries also use this message, but may additionally send BATTERY_INFO.</description>
      <field type="uint8_t" name="id" instance="true">Battery ID</field>
      <field type="uint8_t" name="battery_function" enum="MAV_BATTERY_FUNCTION">Function of the battery</field>
      <field type="uint8_t" name="type" enum="MAV_BATTERY_TYPE">Type (chemistry) of the battery</field>
      <field type="int16_t" name="temperature" units="cdegC" invalid="INT16_MAX">Temperature of the battery. INT16_MAX for unknown temperature.</field>
      <field type="uint16_t[10]" name="voltages" units="mV" invalid="[UINT16_MAX]">Battery voltage of cells 1 to 10 (see voltages_ext for cells 11-14). Cells in this field above the valid cell count for this battery should have the UINT16_MAX value. If individual cell voltages are unknown or not measured for this battery, then the overall battery voltage should be filled in cell 0, with all others set to UINT16_MAX. If the voltage of the battery is greater than (UINT16_MAX - 1), then cell 0 should be set to (UINT16_MAX - 1), and cell 1 to the remaining voltage. This can be extended to multiple cells if the total voltage is greater than 2 * (UINT16_MAX - 1).</field>
      <field type="int16_t" name="current_battery" units="cA" invalid="-1">Battery current, -1: autopilot does not measure the current</field>
      <field type="int32_t" name="current_consumed" units="mAh" invalid="-1">Consumed charge, -1: autopilot does not provide consumption estimate</field>
      <field type="int32_t" name="energy_consumed" units="hJ" invalid="-1">Consumed energy, -1: autopilot does not provide energy consumption estimate</field>
      <field type="int8_t" name="battery_remaining" units="%" invalid="-1">Remaining battery energy. Values: [0-100], -1: autopilot does not estimate the remaining battery.</field>
      <extensions/>
      <field type="int32_t" name="time_remaining" units="s" invalid="0">Remaining battery time, 0: autopilot does not provide remaining battery time estimate</field>
      <field type="uint8_t" name="charge_state" enum="MAV_BATTERY_CHARGE_STATE">State for extent of discharge, provided by autopilot for warning or external reactions</field>
      <field type="uint16_t[4]" name="voltages_ext" units="mV" invalid="[0]">Battery voltages for cells 11 to 14. Cells above the valid cell count for this battery should have a value of 0, where zero indicates not supported (note, this is different than for the voltages field and allows empty byte truncation). If the measured value is 0 then 1 should be sent instead.</field>
      <field type="uint8_t" name="mode" enum="MAV_BATTERY_MODE">Battery mode. Default (0) is that battery mode reporting is not supported or battery is in normal-use mode.</field>
      <field type="uint32_t" name="fault_bitmask" enum="MAV_BATTERY_FAULT">Fault/health indications. These should be set when charge_state is MAV_BATTERY_CHARGE_STATE_FAILED or MAV_BATTERY_CHARGE_STATE_UNHEALTHY (if not, fault reporting is not supported).</field>
    </message>
    ```
  
  - **`<message>` element:** Defines the data structure of a single MAVLink message that can be sent over the wire as a set of `<field>` elements.
    
    - Attributes:
      
      - `<id>`(required): The unique index number of this message (in the example above: 147).
        
        - For MAVLink 1:
          
          - Valid numbers range from 0 to 255.
          
          - The ids 0-149 and 230-255 are reserved for `common.xml`. Dialects can use 180-229 for custom messages.
        
        - For MAVLink 2:
          
          - Valid numbers range from 0 to 16777215.
          
          - All numbers below 255 should be considered reserved.
      
      - `<extensions/>` (optional): This self-closing tag is used to indicate that subsequent fields apply to MAVLink 2 only.
  
  - **`<field>`** element: Encodes one field of the message, specifying its data type and other metadata for display.
    
    - Attributes:
      
      - `type`: The size of the data required to store/represent the data type.
        
        - Fields can be signed/unsigned integers of size 8, 16, 32, 64 bits (`{u)int8_t`, `(u)int16_t`, `(u)int32_t`, `(u)int64_t`). They can also be arrays of the other types - e.g. `uint16_t[10]`.
      
      - `name`: Name of the field (used in code).
      
      - `invalid`: Specifies a value that can be set on a field to indicate that the data is `invalid`: the recipient should ignore the field if it has this value.
        
        - For example, `BATTERY_STATUS.current_battery` specifies `invalid="-1"`, so a battery that does not measure supplied *current* should set `BATTERY_STATUS.current_battery` to `-1`.
