# Tutorial 0: Environment setup 

### Table of Content:

1. [Before_you_start](#Before_you_start)
2. [Euipment_you_need](#Euipment_you_need)
3. [Development_flow](#Development_flow)
4. [Programing](#Programing)
5. [Debug](#Debug)

Mbed OS reference:

https://os.mbed.com/docs/mbed-os/v6.13/mbed-os-api-doxy/group__drivers-public-api-gpio.html

## Basics

#### What is this tutorial for ?

This tutorial is for the robocon junior that are willing and able to take part of  the electronic and progamming of the robocon team 2021~2022. for now we just have the mbed os tutorial, other system like 

#### What we learn after this tutorial

You should be able to develop an robot module controled by a mcu and be able to collerborate with others in the team.

#### What is mbed OS

Mbed OS is an open-source operating system for Internet of Things (IoT) Cortex-M boards: low-powered, constrained and connected. Mbed OS provides an abstraction layer for the microcontrollers it runs on, so that developers can write C/C++ applications that run on any **Mbed-enabled board**. (Not every STM32 MCUs are mbed enabled)

As in user aspect, the Mbed OS platform for debugging and more user friendly for beginners to develop an mcu program from the given library / driver provide from the Mbed. 

#### What are the basic MCU platform we use 

1. Nucleo STM32F446RE

   ![](https://github.com/SamsonChau/Robocon_Training_2021-2022_backup/blob/main/pic/nucleo_f446re.png)

   The most used MCU dev board in most recent Polyu Robocon. This board is good for development as it has all the interface we need as a MCU and the relitivly fast clock speed(180MHz) and relatively large flash storage (512KB).

   However, due to the misuse of the people (mostly short circuit and reverse polarity or overload the LDO) 

   Also this board is the most rare and expensive board within the MCU board we use. 

   We alway use the Mbed Studio to prgram it but if you think Keil studio is more suit u, u may use it also. 

   **Note for this board:**

   1. This board can work on 5v or 3.3v logic level

   2. Each GPIO pin should not draw more than 400mA

   3. **Always power by the PC USB or power by the E5V via Vin pin**

   4. Never ever reverse polarity

   5. Alway keep you work environment clean if you place the board on table directly 

   6. Power every thing before you varify the connection is correct

      

2. Arduinos

   ![](https://github.com/SamsonChau/Robocon_Training_2021-2022_backup/blob/main/pic/different-types-of-Arduino-boards.jpeg)

   The most popular boards for beginer, this board or this type of board with different dimension size, pinout and MCUs you can choose any Arduinos depend on your need.

   These board will be used, if we want to create rapid protype or an module that perform a single task (e.g like read a sensor, or control some relays) . But most of the time,  we will use it because the library was already created by other opensource project, and we do not want to develop a mbed verison of it. 

   Mostly we will use Arduino IDE to program it.  

   **Strength**

   1. Strong opensource backup
   2. Easy to use (Beginner friendly)
   3. Can be connected to near every thing you need

   **Weakness**

   1. Not suitable for very speed dependant task (e.g. multi-BLDC PID control) as the arduino usually 
   2. Not suitable for a very large project since the flash size is lower

   **Common Model**

   * Arduino uno
   * Arduino nano
   * Arduino promini
   * Arduino mega

   **Note for this board:**

   1. This board can work on 5v  logic level only (except for promini may have 3.3v version)
   2. Each GPIO pin should not draw more than 400mA
   3. **Always power by the PC USB or power by the via Vin pin (9V)**
   4. Never ever reverse polarity
   5. Alway keep you work environment clean if you place the board on table directly 
   6. Power every thing before you varify the connection is correct

3. ESP32s

   ![](https://github.com/SamsonChau/Robocon_Training_2021-2022_backup/blob/main/pic/ESP32-Pinout.jpg)

   The new module we added in use in this year as this module is easy to use and more easy for wireless connection and faster than arduino or some stm32s. For now jsut one project using it. 

   For now we are evaluating the usage of this board and may possible add in to the tutorial later

##  Euipment you need (Recommanded)

1. PC with Mbed OS installed

   **usage**

   * program and debug the device

   * check and review the datasheet and circuit you need

2. A mbed enabled STM development board or your stm based setup

   **usage**

   Execue the program and review the output 

3. A Digitial probe 

   **usage**

   * Check the wire connection is secure or not
   * Check the voltage of the test point
   * Check any electroical properties you may want to varify 

4. An oscilloscope (optional if your HW is not verify yet)

   **usage**

   * Check the input and output signal of the certain bus/ output pin
   * Check the Transient of a certain signal

5. A power supply (recommanded)

   **usage**

   If you have some new hardware to test, and **IT IS NOT A MOTOR DRIVER OR SOMETHING WILL DRAW A LOT OF CURRENT (>=3A) ** . We will suggest you may test your circuit with a power supply first. Since you can limit  the amount of current output and prevent any catastrophic failure. But still, if you perform incorrect operation on your device(like short circuit or overvoltage), the magic smoke may happen. 

## Development flow

There have no stright rules for how to develop a electronic device or system. Here only show one of a method to develop a module, you may feel more efficient with other approche. You can use different approche. 

**Steps**

1. Varify the functionality of your system
2. Gether the hardware compoenent you need for this system  
3. Test the system in the test module / test board you have
4. Develop the software part at the same time (review the data sheets and write a first test version program)
5. Test the program with the hardware until you satisfy the result (Not just make it work, but make it stable and easy to use)
6. Test the communication integrate with the system.
7. With a Readme / documentation to let other people can repilcate your project, and make use of it. 
8. Finish the module by install it in the actrual robot and continuously  update & debug the program

## Programing



## Debug

