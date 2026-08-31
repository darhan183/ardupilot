#### Ardupilot Code Layers

---

##### Ardupilot Code Structure:

                                                                            ![](/home/d.arhan/Pictures/Screenshots/2026-08-31-15-12-24-Screenshot%20from%202026-08-31%2015-11-31.png)

- The basic structure of ArduPilot is broken up into 5 main parts:
  
  - **<u>Vehicle Code:</u>** The vehicle directories are the top level directories that define the firmware for each vehicle type.
    
    - There are 6 vehicle types: Plane, Copter, Rover, Sub, Blimp and AntennaTracker.
  
  - **<u>Libraries:</u>** The libraries are shared amongst all vehicle types. These libraries include sensor drivers, attitude and position estimation (aka EKF) and control code (i.e. PID controllers).
  
  - **<u>AP_HAL:</u>** The `AP_HAL layer (Hardware Abstraction Layer)` is how we make ArduPilot portable to lots of different platforms.
    
    - There is a top-level `AP_HAL in libraries/AP_HAL` that defines the interface that the rest of the code has to specific board features.
    
    - There is a `AP_HAL_XXX` subdirectory for each board type, for example, `AP_HAL_ChibiOS` for stm32-based boards, `AP_HAL_ESP32` for ESP32 boards and `AP_HAL_Linux` for Linux based boards.
  
  - **<u>Tool Directories:</u>** The tools directories are miscellaneous support directories.
  
  - **<u>External Support Code:</u>** On some platforms we need external support code to provide additional features or board support. Currently the external trees are:
    
    - **ChibiOS:** The `ChibiOS RTOS` used on `stm32-based boards`.
    
    - **DroneCAN:** The `CANBUS` implementation used in ArduPilot.
    
    - **MAVLink:** The mavlink protocol and code generator.

---

#### Sensor Drivers:

- **<u>Supported Protocols:</u>** I2C, SPI, UART and DroneCAN protocols are supported.
  
  - If you plan to write a new driver, you will likely need to refer to the sensor’s datasheet in order to determine which protocol it uses.

- **<u>I2C:</u>** **Inter Integrated Circuit**
  
  - One master, many slaves.
  
  - A relatively simple protocol which is good for communicating over short-distances (i.e. less than 1m).
  
  - Bus runs at `100kHz` or `400kHz` but the data rate is relatively low compared to other protocols.
  
  - Only 4 pins are required (VCC, GND, SDA, SCL).
    
    <img src="file:///home/d.arhan/Pictures/Screenshots/2026-08-31-16-45-31-image.png" title="" alt="" width="327">

- **<u>SPI:</u>** Serial Peripheral Interfaces.
  
  - One master, one slave.
  
  - 20Mhz+ speed meaning it is very fast especially compared to I2C.
  
  - Only works over short distances (10cm).
  
  - Requires at least 5 pins (VCC, GND, SCLK, Master-Out-Slave-In, Master-In-Slave-Out) + 1 slave select pin per slave.
    
    <img src="file:///home/d.arhan/Pictures/Screenshots/2026-08-31-16-53-11-image.png" title="" alt="" width="357">

- **<u>UART:</u>** Universal Asynchronous Receiver Transmitter
  
  - One master, one slave.
  
  - Character based protocol good for communicating over longer distances compared to I2C and SPI (i.e. 1m).
  
  - Relatively fast at 57Kbps ~ 1.5Mbps
  
  - At least 4 pins required (VCC, GND, TX, RX), plus 2 optional pins (Clear-To-Send, Clear-To-Receive)

- **<u>CAN Bus with UAVCAN:</u>** 
  
  - Multimaster bus, any node can initiate transmission of data when they need to.
  
  - Packet based protocol for very long distances.
  
  - At least 3 pins required (GND, CAN_HIGH, CAN_LOW).
  
  - Point-to-point topology.
  
  - Termination is required at each end of the bus.
    
    <img src="file:///home/d.arhan/Pictures/Screenshots/2026-08-31-16-59-33-image.png" title="" alt="" width="443">

- **<u>FrontEnd/BackEnd Split:</u>** 
  
  ![](/home/d.arhan/Pictures/Screenshots/2026-08-31-17-00-46-image.png)
  
  - The vehicle code only ever calls into the Library’s (aka sensor driver’s) front-end.
  
  - The front-end maintains pointers to each back-end which are normally held within an array named _drivers[].
  
  - On start-up the front-end creates one or more back-ends based either on automatic detection of the sensor or by using the user defined _TYPE params (i.e. RNGFND_TYPE, RNGFND_TYPE2).

- **<u>How and When the driver code is run:</u>**
  
  ![](/home/d.arhan/Pictures/Screenshots/2026-08-31-19-44-58-image.png)

- 








