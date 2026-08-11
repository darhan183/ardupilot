#### <u>Multirotor</u>: [Multirotor - Wikipedia](https://en.wikipedia.org/wiki/Multirotor)

- A multirotor is a rotorcraft with more than two lift-generating rotors separated horizontally.

- An advantage of multirotor aircraft is the simpler rotor mechanics required for flight control.

- They control vehicle motion by varying the relative speed of each rotor to change the thrust and torque produced by each.

- Due to their ease of both construction and control, multirotor aircraft are frequently used in `radio control aircraft` and `Unmanned Aerial Vehicle (UAV)` projects. For example: Tricopter, Quadcopter, hexacopter , octocopter etc.

#### <u>Subsystems of Multirotor</u>: [Drone Flight Controllers: A Comprehensive Guide | Grepow](https://www.grepow.com/blog/what-is-a-drone-flight-controller.html)

1. <b><u>Flight Controller Unit</u></b>:
   
   - UAV are complex systems that rely on several components to achieve stable flight and perform various functions.
   
   - Among these components, <b>the flight controller plays a critical role, acting as the `Brain`</b> of the drone that interprets data from various sensors and translates pilot commands into motor actions.
   
   - A `Flight Controller (FC)` is the central processing unit of a drone.
   
   -  It integrates sensors, software, and communication modules to control the drone’s flight and stability.
   
   - Core responsibilities of a flight controller include:
     
     - <b>Stabilizing flight</b>: Using gyroscopes and accelerometers, the FC ensures the drone remains balanced and level during flight.
     
     - <b>Processing input commands</b>: It interprets commands from the remote control or autopilot system and adjusts the drone’s motors accordingly.
     
     - <b>Navigation</b>: Some flight controllers incorporate GPS for autonomous navigation and return-to-home functionality.
     
     - <b>Data integration</b>: They communicate with external systems such as cameras, payloads, or telemetry modules.
   
   - Flight controllers often run on specialized firmware like Betaflight, ArduPilot, or PX4, which enable features ranging from basic stability to advanced autonomous flight.
   
   - <B><u>The Relationship between Drone Flight Controllers and Related Software</u></B>:
     
     - The flight controller is the physical hardware component that receives sensor data, processes it, and sends commands to the motors.
     
     - The software, often referred to as firmware or flight control software, is the digital brain that runs on the flight controller, interpreting sensor data, executing flight algorithms, and controlling the drone's movements.
     
     - <b>Software Provides Operational Rules:</b> The firmware on the flight controller dictates how it processes sensor inputs, interprets pilot commands, and adjusts motor outputs.
     
     - <b>Customization and Upgrades:</b> Flight controller software allows users to customize flight characteristics (e.g., sensitivity, flight modes) and enables firmware updates for improved performance or added features.
     
     - <b>Communication and User Interface:</b> Software enables users to interact with the FC through configuration tools, tuning parameters, and troubleshooting.
     
     - <b>Autonomy and Advanced Functions:</b> For UAVs with autonomous capabilities, the software handles waypoint navigation, obstacle avoidance, and other complex tasks by processing GPS data and sensor feedback.

2. <b><u>Motors</u>:</b> [Drone Motors Explained: What They Are, Types &amp; Components](https://mechtex.com/blog/basic-of-drone-motor-what-they-are-their-types-and-their-components)
   
   - A drone motor is a specialised electric motor that generates the thrust required to lift the drone.
   
   - It converts electrical energy into mechanical energy and spins the propellers at high speed to create airflow to lift the drone.
   
   - The drone motors works with other components and forms an integrated propulsion system. The key components are:
     
     - <b>Motor(Brushed or Brushless Motor)</b>: Generates rotational force to spin propellers and lift the drone.
     - <b>Propellers</b>: Attach to the shaft of the motor and generate thrust to lift the drone.
     - <b>Electronic Speed Controller</b>: Controls the speed and direction of the drone motor through electrical signals.
     - <b>Flight Controller</b>: Send commands to the ESC based on inputs from sensors and other algorithms for stable flight.
   
   - The drone motor is a DC motor which operates on direct current supplied by the batteries. It works by receiving the signal from the ESC (Electronic Speed Controller).
     
     - These signals convert direct current into a three-phase signal, which creates a rotating magnetic field in the stator.
     - The permanent magnet in the rotor interacts with this magnetic field and starts rotation.
     - Motor speed can vary with the help of PWM signals from the ESC to enable precise control and smooth flight.
   
   - <b>Types of Motors</b>: 
     
     <img src="https://mechtex.com/uploads/images/202401/image_750x_65aa6839d490a.jpg" title="" alt="Types of Drone Motor" width="470">
     
     - <b><u>Brushed DC Motor</u>:</b> This motor consists of stator, rotor, commutator and brushes.
       
       - These motors use carbon brushes and commutators to switch the current within the motor's winding. It energises the rotor winding and creates a magnetic field that causes rotation.
       
       - The brushed DC motors have low efficiency, limited life span, and generate more friction during operation.
     
     - <b><u>Brushless DC Motor</u></b>: This motor consists of stator, rotor and ESC (Electronic Speed Controller). It eliminates the use of brushes and commutators.
       
       - The ESC switch the direction of current and energises the stator coil in a specific sequence.
       
       - It creates a rotating magnetic field in the stator winding. The rotor interacts with the stator winding and causes the rotor to spin.
       
       - BLDC Motor provide high efficiency, high power-to-weight ratio, and low maintenance requirements, which makes them ideal for modern drones.
       
       - BLDC motors are categorised into two types:
         
         1. <b><u>Outrunner Motors</u></b>:  The rotor spins around the stator and generates high torque at low RPM, this design is ideal for drones lifting heavy payloads.
         
         2. <b><u>Inrunner Motor</u></b>: The rotor spins inside the stator winding and generates high RPM at low torque. This makes them ideal for fixed-wing drones where speed is a priority over torque.
     
     - <b><u>Coreless Motor</u></b>: are specialised form of brushed DC motor without an iron core in the rotor.
       
       - They are extremely lightweight motors and offer fast acceleration. It makes them suitable for small or micro drones, which are used for recreational purposes.
     
     - <b><u>Gimbal Motors</u>:</b> are used for stabilising the cameras used in drones.
       
       - These motors offer precise control over camera movement and high torque for effective stabilization.
     
     - <b><u>Tilt-Rotor Motors</u>:</b> are specialised motors that can be rotated to transition between horizontal and vertical flight modes.
       
       - The motor is mounted on the aircraft wing and is capable of tilting upward and downward.
       
       - The tilt-rotor motor allows the aircraft to take off and land vertically like a helicopter.
   
   - <b><u>Key Specification of Drone Motor</u>:</b> 
     
     1. <b><u>KV Rating(RPM per Volt)</u>:</b> The KV rating indicates how many revolutions per minute a BLDC motor will turn per volt of electricity without any load.
        
        - <b>High KV Rating Motors:</b> High KV motors (eg, 1000 KV to 2500 KV) spin faster and are ideal for racing drones where speed is the priority.
        
        - <b>Low KV Rating Motors:</b> Low KV rating motors (eg, 400 to 800 KV) spin slower but provide more torque, which makes them ideal for heavy lift drones that carry heavy payloads.
     
     2. <b><u>Thrust</u>:</b> is the amount of upward force the drone motor can generate with a given propeller.
        
        - The thrust of the motor must exceed the drone's weight for stable flight.
        
        -  A 2:1 thrust-to-weight ratio is recommended for smooth manoeuvring and payload support.
     
     3. <b><u>Propeller Size</u>:</b> The propeller size must match the motor KV, frame size, and battery voltage to optimise drone performance, flight time, and efficiency.
        
        - <b><u>Large Propellers</u></b>: It generates more lift but requires low KV motors.
        
        - <b><u>Small Propellers</u>:</b> Spin faster with a high KV motor and are used for drones where speed is required.
