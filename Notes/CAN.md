### Controller Area Network: https://www.csselectronics.com/pages/can-bus-simple-intro-tutorial

- <b>What is CAN bus ?</b>
  
  - CAN Bus is a communication system used in vehicle/machines to enable `Electronic Control Units(ECU)` to communicate with each other without a host computer. For example, the `CAN bus` enables quick and reliable sharing of information between your car's brakes and engine.
  -  All ECUs are connected on a two-wire bus consisting of a twisted pair: `CAN high` and `CAN low`.
  
  <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-08-11-21-15-11-image.png" title="" alt="" width="274">

- <b><u>CAN Bus DB9 Connector</u>:</b>
  
  - The CAN DB9 (D-sub 9) connector is used to connect a data logger or interface to the CAN bus.
  
  <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-08-11-21-20-44-image.png" title="" alt="" width="361">

- <b><u>CAN Bus Variants</u>:</b> 
  
  | Property              | Low-speed CAN<br/>(Fault Tolerance CAN)                            | Classical CAN 2.0<br/>(High-speed CAN)                   | CAN FD<br/>(Flexible Data-rate)        | CAN XL                                 |
  |:---------------------:|:------------------------------------------------------------------:|:--------------------------------------------------------:|:--------------------------------------:|:--------------------------------------:|
  | Max Baud Rate Speed   | 0.125 Mbit/s                                                       | 1 Mbit/s                                                 | 8 Mbit/s                               | 20 Mbit/s                              |
  | Max Data Payload Size | 8 bytes                                                            | 8 Bytes                                                  | 64 Bytes                               | 2048 Bytes                             |
  | Baud Rate Type        | Fixed                                                              | Fixed                                                    | Variable(Faster Data Field)            | Variable(Higher Rates)                 |
  | Key Feautres          | Fault tolerant operation, continue even if one bus line is damaged | Low cost, robust error detection, most commonly deployed | Increased payload, speed & reliability | Increased payload, speed & reliability |

- <b><u>CAN Physical & Data Link Layer</u>:</b>  The controller area network is described by a `data link layer`  & `physical layer`.
  
  - <u><b>Physical Layer</b></u>: The CAN bus `physical layer` defines cable types, electrical signal level, node requirements etc. For example,
    
    - <u>Baud Rate:</u> Nodes must be connected via a two-wire bus with baud rates up to 1 Mbit/s (Classical CAN) or 8 Mbit/s  (CAN FD)
    
    - <u>Cable Length:</u> Maximal CAN cable lengths should be between 500 meters (125 kbit/s) and 40 meters (1 Mbit/s).
  
  - <u><b>Data Link Layer</b></u>: The CAN bus `Data Link Layer` defines CAN Frame Formats, error handling, data transmission & helps ensure data integrity.
    
    - For example, the data link layer specifies:
      
      - <u>Frame Formats</u>:Four types (data frames, remote frames, error frames, overload frames) and 11-bit/29-bit identifiers.
      
      - <u>Error Handling</u>: Methods for detecting/handling CAN errors including CRC, acknowledgement slots, error counters.
    
    <img title="" src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-08-11-21-49-31-image.png" alt="" width="428">

- 


