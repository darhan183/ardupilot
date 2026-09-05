#### <u>Activity 4</u>

##### Prepared By: Arhan Dayal

##### Date: 5th September 2026

---

- <u>**Build ArduCopter:**</u> 
  
  ```bash
  ./waf configure --board CubeBlack
  ./waf copter
  ```
  
  - `./waf configure` command configures the build environment. This command should be called only once or when you want to change a configuration option.
  
  - One configuration often used is the `--board` option to switch from one board to another one.
  
  - As shown in example given below:
    
    ```bash
    ./waf configure --board skyviper-v2450
    ./waf copter
    ```
  
  - The `arducopter` binary should appear in the `build/<board-name>/bin` directory.

- **<u>List Available Boards:</u>** To get the list of supported boards on ArduPilot, write the command given below,
  
  ```bash
  ./waf list_boards
  ```

- **<u>List of available vehicle types:</u>** Below is the list of most common vehicle build targets:
  
  ```bash
  ./waf copter                            # All multirotor types
  ./waf heli                              # Helicopter types
  ./waf plane                             # Fixed wing airplanes including VTOL
  ./waf rover                             # Ground-based rovers and surface boats
  ./waf sub                               # ROV and other submarines
  ./waf antennatracker                    # Antenna trackers
  ./waf AP_Periph                         # AP Peripheral
  ```

- **<u>Clean The Build:</u>** Commands `clean` and `distclean` can be used to clean the objects produced by the build.
  
  - `./waf clean` removes only the compiled object files for the current board while keeping your configuration settings.
  
  - `./waf distclean` deletes everything for all boards, including all saved configuration information.

- To Upload the build in the flight controller, write the following command,
  
  ```bash
  ./waf copter --upload
  ```

- Below is the command list which we need to do to upload the build in the `flight controller`
  
  ```bash
  ./waf distclean
  ./waf configure --board BOARD_NAME
  ./waf copter
  ./waf copter --upload
  ```

- Below is the command list for the build of AP_Periph,
  
  ```bash
  ./waf distclean
  ./waf configure --board AP_Periph_Board_Name
  ./waf AP_Periph
  ```

---

- When you run the above command, to verify whether command worked or not, go to the folder given below,
  
      Ardupilot/Build/BoardName/bin

- At this location you will see the following files,
  
  <img src="file:///home/d.arhan/Pictures/Screenshots/2026-09-05-18-23-03-image.png" title="" alt="" width="601">

- If we want to upload firmware through **Mission Planner**, we use the **`.apj`** file. This file contains the compiled ArduPilot firmware generated from the vehicle's source code (`.cpp` and `.h` files) and is used by the bootloader to program the flight controller.

- A `.hex` file contains the compiled firmware and can be uploaded directly to the microcontroller using a programmer such as ST-Link based flashing tools.
