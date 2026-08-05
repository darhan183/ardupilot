#### <u>SITL Simulator(Software In The Loop)</u>

##### What is ArduPilot SITL ?

- ArduPilot SITL is a simulation environment that lets us run the real ArduCopter or ArduPlane flight controller software on our computer without any physical drone.

- When running in SITL the sensor data comes from a flight dynamics model in a flight simulator.

- ArduPilot has a wide range of vehicle simulators built in, and can interface to several external simulators. This allows ArduPilot to be tested on a very wide variety of vehicle types. For example, SITL can simulate:
  
  - multi-rotor aircraft
  
  - fixed wing aircraft
  
  - ground vehicles
  
  - underwater vehicles
  
  - camera gimbals
  
  - antenna trackers

##### <u>SITL Architecture:</u>

<img title="" src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-06-22-12-34-00-image.png" alt="" width="482">

         Fig 1: SITL Architecture

- The SITL system is made up of four main software components, each components having their own specific role. They all run simultaneously and communicate with each other over the network.

- The 4 software components are given below:
  
  1. <b><u>ArduPilot/Flight Controller</u></b>:- is the real autopilot software containing the same code which runs of simulator as well as physical flight controller board.
     
     - It reads sensor data, runs flight algorithms & outputs motor commands.
     
     - In SITL mode it runs as a regular program on our PC instead of on embedded hardware.
     
     - It communicates via virtual serial ports mapped to TCP or UDP network connections.
  
  2. <b><u>Physics Simulator</u></b>: Receives motor commands from the flight controller.
     
     - Calculates the resulting drone motion using physics equations.
     
     - Sends back fake sensor readings: GPS position, accelerometer, barometer, compass, etc.
     
     - `sim_multicopter.py`is used for `copters`; `JSBSim` is used for fixed-wing planes.
  
  3. <b><u>FlightGear:</u></b>  An optional 3D flight simulator that provides a visual display of the drone flying.
     
     - Receives position and attitude data from physics simulator.
     
     - It has no effect on the actual simulation.
  
  4. <b><u>Ground Control Stations(The Pilot's Console)</u></b>:- There are the apps a human operator uses to monitor & control the drone, which are given below: 
     
     - Mission Planner
     
     - MAVProxy
     
     - Other GCS like QGroundControl and similar app can connect via MAVProxy on UDP port 14550.

- ##### <u>How The Components Communicate:-</u>
  
  - All four components run on the same machine or local network & talk to each other using standard network protocols: `TCP & UDP` . Each piece of software listens on a different port number to avoid mixing up messages.
  
  - The communication is the closed loop between the `Flight controller` & `the physics simulator`, this loop runs continuously.
  
  - According to architecture we can see that, 
    
    - `Flight Controller` sends the `Motor Commands(UDP 5502)` to the `Physics Simulator`.
    
    - `Physics Simulator` sends the `sensor data(UDP 5501)` to the `Flight Controller`.
      
      ###### <u>NOTE:-</u> UDP is used here instead of TCP because speeds matter more than guranteed delivery.

---

##### <u>Setting Up SITL on Linux</u>:-  [Setting up SITL on Linux &mdash; Dev documentation](https://ardupilot.org/dev/docs/setting-up-sitl-on-linux.html)[Setting up SITL on Linux &mdash; Dev documentation](https://ardupilot.org/dev/docs/setting-up-sitl-on-linux.html)

- <b><u>Installation Steps:-</u></b> [Setting up the Build Environment (Linux/Ubuntu) &mdash; Dev documentation](https://ardupilot.org/dev/docs/building-setup-linux.html#building-setup-linux)
  
  - Click on the above link to get on the installation web page.
    
    ###### <u>NOTE:-</u> You should have an admin access to continue the installation process which you can get it from IT Department.
  
  - For `Ubuntu` users,
    
    - Write all the following commands in the terminal,
      
      ```bash
      sudo apt-get update
      
      sudo apt-get install git
      
      sudo apt-get install gitk git-gui
      ```
    
    - Youtube link:- [Git setup - YouTube](https://youtu.be/G1Kc-1aF8HI?si=DG-dW4z12CqTBop3)
    
    - <b>Clone ArduPilot Repository:-</b> 
      
      - Cloning is git's term for making a local copy of a remote repository.
      
      - Copy the following git website link to access the ardupilot repo,
        
        ```bash
        https://github.com/ArduPilot/ardupilot.git
        ```
      
      - After accessing the ArduPilot repo, do the following steps
        
        1. First create your git account.
        
        2. After creating the account, fork the ardupilot repo.
        
        3. After fork of the ardupilot, you copy the URL of fork ArduPilot repo which will look like this,
           
           ```bash
           https://github.com/darhan183/ardupilot.git
           ```
      
      - Now on terminal, copy paste the following command,
        
        ```bash
        git clone --recurse-submodules https://github.com/your-github-userid/ardupilot
        cd ardupilot
        ```
      
      - Youtube link:- [Cloning the repo - YouTube](https://youtu.be/kAli2y2-n-M?si=wEnqU7_gievb5aDj)
    
    - After getting inside the ardupilot directory, write the following command
      
      ```bash
      Tools/environment_install/install-prereqs-ubuntu.sh -y
      ```
    
    - Reload the path (log-out and log-in to make it permanent):
      
      ```bash
      . ~/.profile
      ```
    
    - Now your SITL installed. 

---

##### <u>Start SITL Simulator:- </u>

- To start the simulator first change directory to the vehicle directory. For examples below, for the Multicopter code change to `ardupilot/ArduCopter` :
  
  ```bash
  cd ardupilot/ArduCopter
  ```

- Then start the simulator using `sim_vehicle.py`. The first time you run it, you should use the `-w` option to wipe the virtual EEPROM of user changed parameters and load the default parameters for your vehicle.
  
  ```bash
  sim_vehicle.py -v ArduCopter -f quad --console --map -w
  ```
  
  ###### <u>NOTE</u>:- This command is for accessing the copter.

---

##### <u>Using SITL</u>:- [Using SITL &mdash; Dev documentation](https://ardupilot.org/dev/docs/using-sitl-for-ardupilot-testing.html)

- Youtube Link:- https://youtu.be/Ewh0fKGEJL4?si=VL2KIDWGKnD4VEOt

- ###### <u>Using sim_vehicle.py</u>:-
  
  - A startup script, `sim_vehicle.py` is provided to automatically build the SITL firmware version for the current code branch, load the simulation models, start the simulator, setup environment and vehicle parameters, and start the MAVProxy GCS.
  
  - To start the `SITL` write the following command,
    
    ```bash
    cd ardupilot/ArduCopter
    ```
    
    ```
    sim_vehicle.py -v ArduCopter -f quad --console --map
    ```

- Following tabs will open when you write this command,
  
  <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-06-22-17-16-24-image.png" title="" alt="" width="588">

- To get help to controll `SITL` just type the following command,
  
  ```bash
  sim_vehicle.py --help
  ```

- ##### Selecting a vehicle/frame type:-
  
  - You can select the vehicle type if starting from any directory by starting the simulator calling `sim_vehicle.py` with the `-v` parameter.
    
    ```bash
    sim_vehicle.py -v ArduCopter --console --map
    ```
  
  - The frame type can also be changed with `-f` parameter.
    
    ```bash
    sim_vehicle.py -v ArduPlane -f quad --console --map
    ```
  
  - To know the frame types & vehicle type just write the help command.
    
    <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-06-22-18-14-22-image.png" title="" alt="" width="364">

- ##### Setting Vehicle Start Location:-
  
  - You can start the simulator with the vehicle at a particular location by calling **sim_vehicle.py** with the `-L` parameter and a named location in the [ardupilot/Tools/autotest/locations.txt](https://github.com/ArduPilot/ardupilot/blob/master/Tools/autotest/locations.txt) file.
  
  - For example, to start Copter in *Ballarat* (a named location in locations.txt) call:
    
    ```bash
    cd ArduCopter
    sim_vehicle.py -L Ballarat --console --map
    ```

- ##### Loading a different default parameter set:-
  
  - A set of default parameters are automatically selected by the frame type.
  
  - We can select a different parameter file by using the `--add-param-file=` option, instead of manually loading them via the GCS after the simulation starts:
    
    ```bash
    sim_vehicle.py -v ArduPlane --console --map --add-param-file=<path to file>
    ```
  
  - The default parameter set emulates the parameter default values contained in the firmware.These are used at startup for parameters, unless the user has previously changed them for a parameter, in which case those changed values are used.
  
  - In SITL, these changes are stored in a file in the simulation startup directory named `eeprom.bin`.
  
  - If you wish to startup with only the default values, either that file can be erased, or the `-w` options can be used.
    
    ```bash
    sim_vehicle.py -v ArduPlane --console --map --add-param-file=<path to file> -w
    ```

- ##### Using Real Serial Devices:-
  
  - It is useful to use a real serial device in SITL. This makes it possible to connect SITL to a real GPS for GPS device driver development.
  
  - To use a real serial device you can use a command like this:
    
    ```bash
    sim_vehicle.py -A "--serial2=uart:/dev/ttyUSB0" --console --map
    ```
  
  - We can set additional parameters on the uart in the connection string, so for instance to use a device on SERIAL1 at 115k baud only, specify:
    
    ```bash
    sim_vehicle.py -v ArduCopter -A "--serial1=uart:/dev/ttyUSB0:115200" --console --map
    ```
  
  - it pass the `–serial1` argument to the ardupilot code, telling it to use `/dev/ttyUSB0` instead of the normal internal simulated GPS with buad rate of `115200`
  
  - To understand more about the SITL Serial Port mappings follow the given link,
    
    [UARTs and the Console &mdash; Dev documentation](https://ardupilot.org/dev/docs/learning-ardupilot-uarts-and-the-console.html#learning-ardupilot-uarts-and-the-console)
  
  - Any of the 8 UARTs can be configured in this way, using serial0 to serial7.

---

##### Copter SITL Tutorial:-

- Preconditions:- SITL should be set up on your system & that you have started SITL using `--map` & `--console` options.

- <b><u>Taking Off</u></b>:- 
  
  - The steps to start the takeoff is given below,
    
    1. First change the mode into `GUIDED` mode by using the given command,
       
       ```bash
       mode guided
       ```
    
    2- Then arm the motors,
    
    ```bash
    arm throttle
    ```
    
    3- Then call the takeoff command,
    
    ```bash
    takeoff 40
    ```
    
       <b>NOTE:-</b> takeoff 40 means your copter will gain the altitude till 40 meters during takeoff, altitude can also be changed.
    
    ```bash
    mode guided
    arm throttle
    takeoff 40
    ```
    
    - Copter should take off to an altitude of 40 metres and then hover while it waits for the next command.
    
    - Attempting to takeoff when the vehicle is not armed. This can happen if you call `takeoff` too slowly after `arm throttle`, or if the vehicle fails pre-arm checks.

- ##### Guiding The Vehicle:-
  
  - After takeoff, right click on the map where you want to go, select `Fly to` & then enter the target altitude.
  
  <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-06-22-19-38-35-image.png" title="" alt="" width="401">
  
  <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-06-22-19-38-42-image.png" title="" alt="" width="401">
  
  - You can also enter the target position manually on the command line using the following command.
    
    ```bash
    guided LAT LON ALTITUDE
    ```
  
  - To get `lat, lon` you just need to do the `Left Click` on the map where you want your copter to fly, after that copy the longitude & latitude and paste it to the command
    
    ![](/home/darhan/snap/marktext/9/.config/marktext/images/2026-06-22-19-45-18-image.png)
    
    ```bash
    guided -35.36392632 149.16351793 50
    ```
    
    where -35.36392632 is LAT
    149.16351793 is LON
    50 is Altitude
    
    

- ##### Flying a Mission:-
  
  - You can load a mission at any time using the `wp load` command. After you’ve taken off the current mission will start as soon as you change to `AUTO` mode.
  
  - The example below shows how to load and start one of the test missions, skip to the second waypoint, and loop the mission:
    
    ```bash
    wp load ardupilot/Tools/autotest/Generic_Missions/CMAC-circuit.txt
    mode auto
    wp set 2
    wp loop
    ```
  
  - If you want to create a waypoint mission, this is most easily done on the map:
    
    1. Right-click on the map and then select  Mission -> Draw.
       
       <img src="file:///home/darhan/snap/marktext/9/.config/marktext/images/2026-06-22-20-00-20-image.png" title="" alt="" width="354">
    
    2. Left-click on the map where you want the points to appear.
    
    3. When you’re done, you can loop the mission by right-clicking on the map and selecting  Mission -> Loop.
    
    4. If you want to save these waypoints then write the command given below,
       
       ```bash
       wp save filename.txt
       ```
       
       5- After saving the waypoints, if you want to load the exact same waypoint then
       
       ```bash
       wp load filename.txt
       ```
  
  - But if the mission is not starting without giving `TAKEOFF` command and you change your mode into `AUTO` mode, then change the parameter which is given below,
    
    - Change `AUTO_OPTIONS = 0` to `AUTO_OPTIONS = 3`
    
    - Below is the given command of how to change the parameter,
      
      ```bash
      param set AUTO_OPTIONS = 3
      ```
    
    - To verify is the parameter change, write the following command
      
      ```bash
      param show AUTO_OPTIONS
      ```
  
  - After that you will be able to start the mission without giving the `TAKEOFF` command, just change the flight mode into `AUTO` mode.
  
  ---
  
  ###### This `AUTO_OPTIONS` parameter is a range of options that can be applied to change auto mode behaviour, the options are given below:
  
  1. Allow Arming, allows the copter to be armed in Auto.
  
  2. Allow Takeoff Without Raising Throttle, allows takeoff without the pilot having to raise the throttle.
  
  3. Ignore pilot yaw overrides the pilot's yaw stick being used while in auto.
  
  | BIT | Meaning                                |
  |:---:|:--------------------------------------:|
  | 0   | Allow Arming                           |
  | 1   | Allow takeoff without raising throttle |
  | 2   | Ignore Pilot Yaw                       |
  | 7   | Allow weathervaning                    |
