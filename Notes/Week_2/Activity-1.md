#### Activity-1

##### Prepared By: Arhan Dayal

##### Date: 5th September 2026

---

##### What is hwdef file?

- Full Form of `hwdef` is `Hardware Definition`.

- In this file we read & describe the pins which are allocated inside the `Flight Controller`. 

#### Read The `hwdef` files:

- Follow the steps given below to read the the `hwdef` (Hardware Defininition) file of `CubeOrange`:
  
  1. Go to Ardupilot folder.
  
  2. Inside the Ardupilot folder go to Libraries/AP_HAL_ChibiOS/hwdef/CubeOrange/hwdef.dat.
  
  3. Also open the hwdef-bl.dat file in same folder which is described above.
  
  <img src="file:///home/d.arhan/Pictures/Screenshots/2026-09-05-13-08-50-image.png" title="" alt="" width="743">

- **<u>hwdef.dat:</u>** This file is used to define the Flight Controller's pin layout, serial port assignments, SPI device table, and other hardware configurations.

- **<u>hwdef-bl.dat:</u>** This file is created to define the bootloader hardware configuration, including USB settings, firmware upload ports, LED pins, flash memory settings, and other resources required during the boot process.

- **<u>Serial Orders:</u>** In `hwdef.dat` or `hwdef-bl.dat` file we see that there is `SERIAL ORDER OTG1 USART2 USART3 UART4 UART8 UART7 OTG2` line in the file. The `SERIAL_ORDER` line specifies the order of the UART and USB ports. ArduPilot uses this order to assign logical serial port numbers (`SERIAL0`, `SERIAL1`, etc.) to the physical interfaces.

---

**<u>Difference between two flight controller board:</u>**

| Feqtures                                 | CubeOrange                                      | MatekF405                          |
| ---------------------------------------- | ----------------------------------------------- | ---------------------------------- |
| Crystal Frequency                        | Oscillator Hz 24000000                          | Oscillator Hz 8000000              |
| Flash Size in KB                         | 2048                                            | 1024                               |
| CAN_FD Supported                         | 8MBits/s                                        | Not supported                      |
| I2C Bus                                  | There are two I2C Buses                         | Only one I2C Bus                   |
| Serial Order                             | OTG1, USART2, USART3, UART4, UART8, UART7, OTG2 | OTG1, USART3, UART4, USART5, UART5 |
| GPS                                      | There are two GPS                               | Doesn't contain any GPS            |
| IOMCU(Input Output Microcontroller Unit) | Only One                                        | No IOMCU                           |
| Flash_Reserve_Start in KB                | 128                                             | 64                                 |
| HAL_Storage_Size                         | 32768                                           | 15360                              |
