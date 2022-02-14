# Robocon Junior Training 2021/2022
By Samson Chau 

## Objectve 

After this training you should understand the basics of:

* The use of Git
* C++ programing (MCU)
* Able to create an application of Mbed 
* Understand the basic IOs and communication protocal we use 
* Have a brief understand of PID
* Understand the command electronic compoenent we use.
* Basic soldering 

## Compoenents

### Electronics 

1. Electronic Rapid Prototyping & Basic Testing
2. Passive compoents and system wiring
3. Diodes && fuse
4. yet to update

### MCU programing (Mbed)

1. BasiC 
2. Extra_Advanced_C
3. Basic IO
4. pwm
5. uart
6. CAN
7. PID

### other Important

1. Git Basic
2. Git Gud

## Task for This year 

In Polyu Robocon the task of this year for junior will list as follow:

#### 1. pneumatic module 

Description:
![](https://github.com/SamsonChau/Robocon_Training_2021-2022_backup/blob/main/pic/16_channel_relay.png)
We will need a CAN BUS interface MCU to receive the command of the Master controller and control the SMC solinoid with Relay.

The module should be able to satisfly the requirement:

1. Can Receive the  CAN BUS command (id 0x50, 1M Bps) 
2. Can control different combination in 16 channel relay (GPIOs)  with a single command 
3. A manual debug button and a state Led to display the state of the module.
4. the extact command can be customize your self

Platform: Arduino IDE 

Software material:

​	CAN BUS Module library:https://github.com/autowp/arduino-mcp2515

​	Arduino ide: https://www.arduino.cc/en/software

​	simular tutorial: https://robojax.com/controlling-16-channel-relay-module-using-arduino

Module :

* CAN_BUS MCP2515 module

* Arduino mega 2560

* 16 channel Relay 

  

#### 2. c620 and m3508 motor driver tuning 

c620-m3508 lib: https://github.com/PolyU-Robocon/C620-C610-controller-PID-code-with-Mbed-STM32-

Description:

We will needed to use this library to control the DJI c620 and m3508 actruators with m3508. Each motor module have to tune seperatly to achieve the maximum performance of the module and prevent damge of the robot mechism 

Content 

yet to update,



#### 3. R1 VESC control and Tuning 

As we needed to handle vesc for th fly wheel shooter this year, we needed to have some kind of control of vesc by the MCU or ROS system. 

![](https://github.com/SamsonChau/Robocon_Training_2021-2022_backup/blob/main/pic/vesc_tool.png)
The Content of this task will be make the following approche work  with VESC:

1. Test the ROS USB Driver and connect the VESC via ROS
2. Control the VESC via CAN BUS from MCUs

(optional)

3. Control a USB to serial device and Connect the  

For the someone who handle this task you should at least

* unterstand and able to write a ros node

* have VESC tools and able to config it 

* know the basics of C++ 

  or 

* Know the MCU prgramming of Mbed and Arduino

* Know the CAN-BUS communication protocal 

* have VESC tools and able to config it 

* Have STM32 or arduino to can bus hardware

Materials Support: https://github.com/SamsonChau/Motor-Driver-Material

#### R1 Pitch Control 

In this year we have a new motor to control the pitch of the shooter with the new rmd motor

material: https://github.com/SamsonChau/Motor-Driver-Material/tree/main/RMD_motor

Model RMD L 9015 ![](https://github.com/SamsonChau/Robocon_Training_2021-2022_backup/blob/main/pic/RMD_L_9015.jpg)

  





A Driver of ROS and STM needed to be satisfied the following requirement:

1. Can Control the Position, Velocity and Torque of the motor 
2. Can monitor the real time Posirion, Velocity, Current and PID setting of the ,motor 
3. Can change parameter and set zero at  runtime 
