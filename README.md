# Cheap Custom ESP32 Drone

I designed a custom drone flight controller PCB based on the ESP32-WROOM-32. The board uses USB-C charging, regulated 3.3 V power delivery, and onboard IMU sensing using the MPU-6050. The PCB is currently being fabricated, and I am actively working on automatic stabilization code for my motors.


## Overview
The goal of this project was to design a low-cost, custom flight controller PCB for a quadcopter drone while gaining hands-on experience with power management, embedded hardware design, and PCB layout. Instead of using a real controller, I wanted to use a free app that can be downloaded from the App store since ESP32 supports Wifi and Bluetooth.

The design is organized into three main parts:
- Power management
- ESP32 control and programming
- Motors and sensors


## Power Management
Power management was a major focus of this design. The board uses a battery charging circuit along with a voltage regulator to safely power all components. The incoming voltage is regulated to 3.3 V using the MIC5219-3.3YM5, which supplies power to both the ESP32 and the MPU-6050.

When the board is connected to USB-C, the voltage passes through a Schottky diode (SS34) and reaches the gate of an AO3401A P-channel MOSFET. This turns the MOSFET off and disconnects the battery (+VBatt) from the system, allowing the drone to run directly from USB power while the battery is charging.

Battery charging is handled by the TP4056, which safely manages the LiPo charging process. Two LEDs connected to the TP4056 indicate whether the battery is charging or fully charged. When the USB cable is unplugged, the MOSFET automatically turns on, reconnecting the battery so it can power the system. Decoupling capacitors are placed throughout the power rails to reduce noise and improve voltage stability.


## ESP32 Control and Programming
The +VBUS line is routed before the voltage regulator so it can supply 5 V power to the CP2102 USB-to-UART converter, which requires a higher voltage than the ESP32.

The USB data lines (D+ and D−) carry compiled code from the computer to the CP2102. The CP2102 converts this USB data into UART signals (TX and RX) that are sent to the ESP32. The RTS and DTR signals from the CP2102 pass through a 2N7002DW MOSFET, which automatically controls the EN and BOOT pins of the ESP32 during programming. This allows the ESP32 to enter firmware upload mode automatically and return to normal operation later without manual button presses.

The ESP32 acts as the main controller of the drone. It processes sensor data, controls motor outputs, and manages the overall behavior of the system. Orientation data from the MPU-6050 is used to adjust motor speeds for stabilization. I also added a UART2 (RXD2/TXD2) connector for future expansion and a four-channel SRV connector for motor control signals. Decoupling capacitors here on the 3.3 V rail help minimize noise from switching components.


## Motors and Sensors
The MPU-6050 is used as a 6-axis IMU to measure acceleration and angular velocity. It communicates with the ESP32 over I²C using the SDA and SCL lines, with pull-up resistors included to ensure reliable communication. The sensor operates at 3.3 V and is powered from the regulated supply.

The MPU-6050 does not control the motors directly. Instead, it sends motion data to the ESP32, which determines how each motor should respond to keep the drone stable. The motors are powered directly from the battery (+VBatt) because they require higher current than the logic circuitry.

Each motor is driven using an SI2302 N-channel MOSFET, allowing low-power control signals from the ESP32 to switch high-current motor loads. The MOSFET gates are driven by the ESP32, the sources are connected to ground, and the drains connect to the motors and battery supply. Since motors generate voltage spikes when switching, flyback diodes are included to protect the MOSFETs and surrounding components.


## Current Status
- Schematic and PCB design completed
- All ERC and DRC checks passed
- PCB fabrication and soldering the components in progress
- Automatic stabilization firmware under development


## Future Work
- Adding object-tracking cameras
- 3D Mapping using SLAM
- Perform flight testing and tuning
- Rev B PCB improvements based on test results
