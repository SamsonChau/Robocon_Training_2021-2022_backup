# Advanced Tutorial 2 - Communication Protocol (CAN) and RoboMaster motor features

## Communication Protocol(CAN)
### What is CAN
CAN, Controller area network, is an electronic communication bus defined by the ISO 11898 standards. Those standards define how communication happens, how wiring is configured and how messages are constructed, among other things. Collectively, this system is referred to as a CAN bus.
![](https://i.imgur.com/16oIj5m.png)
### Principle of CAN
Differential Signal
![](https://i.imgur.com/PfL9kJR.png)


From the above graph, we can observe that, CNA is different from normal signal which using  5V and 0V to represent logic 1 and logic 0. It use differential signal, when there is a difference in voltage, it mean 0 and otherwise meaning 1.
ADV of Differential signal:
**Noise tolerance
Accurate timing positioning**

#### Data Frame
Stanard CAN message (11-bit identifer)
![](https://i.imgur.com/xi9v8vW.png)

Extended CAN message(29-bit identifier)
![](https://i.imgur.com/Ok3YeQc.png)
A CAN data frame can be divided into 8 parts:
1. SOF:Start of frame
2. CAN ID
3. RTR
4. Control
5. Data
6. CRC
7. ACK
8. End of frame

Normally, we will only focusing on three parts: CAN ID, Control and Data.

**CAN ID**: indicating where the data frame should be sent to and who should receiving it.

**Control**: Inform the length of data in bytes (0-8).

**Data**: Containing Actual data value, e.g. current for motor.

## Hardware connection  of the can 
To connect your MCU / PC to can bus you will need to connect to a can tranceiver to send the differencial signal on the bus, for STM32F446RE/ STM32F103RB it has the internal can rxtx controller to control the signal to the can server. But for arduino and esp32, the MCU didnot have can tx rx controller, therefore we use the MCP2515 SPI to canbus controller to connect the can tranceiver via SPI. The connection is listed as follow:

![](https://github.com/SamsonChau/Robocon_Training_2021-2022_backup/blob/main/pic/canbus-connect.png)

There have some common ic otherthan TJA1050 can be used as can tranceiver:
1. TJA1050 (very common, cheap, easy to find, we have a lot in lap)
2. TJA1051/1052 (newwer version of TJA1050)
3. MCP2551 (very common and cheap sometimes, the performance simular to tja1050)
4. SN65HVD230 (very common and not so cheap sometimes, use in jetsons)
5. ISO1050 (have power isolation, but need to be powered both from side)

### Mbed STM32 CAN config

mbed refence: https://os.mbed.com/docs/mbed-os/v6.15/apis/other-driver-apis.html

### CAN functions
#### Initialize CAN
We have create the can object for you. For below example, we are using can1 port to send a msg to can 2 port.
```c

#if !DEVICE_CAN
#error [NOT_SUPPORTED] CAN not supported for this target
#endif

#include "mbed.h"

Ticker ticker;
DigitalOut led1(LED1);
DigitalOut led2(LED2);
/** The constructor takes in RX, and TX pin respectively.
  * These pins, for this example, are defined in mbed_app.json
  */
CAN can1(MBED_CONF_APP_CAN1_RD, MBED_CONF_APP_CAN1_TD);
CAN can2(MBED_CONF_APP_CAN2_RD, MBED_CONF_APP_CAN2_TD);
char counter = 0;

void send()
{
    printf("send()\n");
    if (can1.write(CANMessage(1337, &counter, 1))) {
        printf("wloop()\n");
        counter++;
        printf("Message sent: %d\n", counter);
    }
    led1 = !led1;
}

int main()
{
    printf("main()\n");
    ticker.attach(&send, 1);
    CANMessage msg;
    while (1) {
        printf("loop()\n");
        if (can2.read(msg)) {
            printf("Message received: %d\n", msg.data[0]);
            led2 = !led2;
        }
        ThisThread::sleep_for(200);
    }
}

```
CAN object have been defined for you with the help of Mbed OS Driver. You only need to use this define the pinout and call function to start CAN.

###  CAN Message 

```shell
CANMessage	(	unsigned int 	_id,
				const char * 	_data,
				unsigned char 	_len = 8,
				CANType 		_type = CANData,
				CANFormat 		_format = CANStandard 
)
```

Creates [CAN](https://os.mbed.com/docs/mbed-os/v6.15/mbed-os-api-doxy/classmbed_1_1_c_a_n.html) message with specific content.

- Parameters

  _id = Message ID

  _data = Mesaage Data

  _len = Message Data length

  _type = Type of Data: Use enum CANType for valid parameter values

  _format = Data Format: Use enum CANFormat for valid parameter values

Definition at line [90](https://os.mbed.com/docs/mbed-os/v6.15/mbed-os-api-doxy/_interface_c_a_n_8h_source.html#l00090) of file [InterfaceCAN.h](https://os.mbed.com/docs/mbed-os/v6.15/mbed-os-api-doxy/_interface_c_a_n_8h_source.html) in Mbed os 



#### Sending CAN message

The id of the message have been set for you as 0x200.

If we want to run the motor, we have have to add message to our CAN transceiver(Tx). We can use this function

```cpp
can1->write(TxMessage);
```

This is an example to define the can message and send it via can 1:

```c
CANMessage TxMessage;
  TxMessage.id = CMD_ID_1;           //Common ID of c620 
  TxMessage.format = CANStandard; //Standard Frame
  TxMessage.type = CANData;       //Data 
  TxMessage.len = 8;              // Size = 8 byte
  
  // Motor Current Value Range from -16384 to 16384
  // Motor 1 Current Control
  TxMessage.data[0] = current1 >> 8; // data1;
  TxMessage.data[1] = current1;      // data2;
  // Motor 2 Current Control
  TxMessage.data[2] = current2 >> 8; // data3;
  TxMessage.data[3] = current2;      // data4;
  // Motor 3 Current Control
  TxMessage.data[4] = current3 >> 8; // data5;
  TxMessage.data[5] = current3;      // data6;
  // Motor 4 Current Control
  TxMessage.data[6] = current4 >> 8; // data7;
  TxMessage.data[7] = current4;      // data8;
  
  can1->write(TxMessage); // sent a message
```
Assuming we are using 4 motor, we can assign different current value to each motor. As we are using CAN1 port, we are going to pass the address of hcan1(&hcan1) in to the function. The current value to be sent to the motor can be seen as the torque of the motor.
#### Retrieving CAN message
We have created 4 variable(Assuming we are using four motor) for you to use, 
motor1_stat, motor2_stat, motor3_stat, motor4_stat, all the variable as shown below.

```c
  can1->read(rxmsg);
  int16_t actual_local_position = (int16_t)(rxmsg.data[0] << 8) | rxmsg.data[1];
  int16_t actual_velocity       = (int16_t)(rxmsg.data[2] << 8) | rxmsg.data[3];
  int16_t actual_current        = (int16_t)(rxmsg.data[4] << 8) | rxmsg.data[5];
```
Parameter: address of the can object. 
As we are using CAN1 port on our board, we are giving the address of the CAN1 object to this function.

To assign the received data to varaible motor1_stat etc. Use...
```c
void c620::c620_read()
```

### Example code
C620-C610-controller-PID-code-with-Mbed-STM32- https://github.com/PolyU-Robocon/C620-C610-controller-PID-code-with-Mbed-STM32-

The code below shows you how to controll the c620 motor using the prebuild library at the same time. This will rotate your motor in alternate the position / rpm of the motor after u press the user button

```c
//In main.cpp
#include "m3508.h"
#include "mbed.h"
#include <cstdio>

CAN can1(PA_11, PA_12, 1000000);
m3508 _m3508;

DigitalIn upbutton(PC_13); // User Button
DigitalOut led(LED1);
bool state = false;

//steps and delay counter
Timer t2;
int step = 0;
int counter = 0;

int main() {
  _m3508.m3508_init(&can1);

  // set the PID term of the 3 Ring PID loop
  // use c610 +2006 as example
  // suggest velocity mode setting 
  //_m3508.set_i_pid_param(0, 1.7, 0.0, 0.000005); // Torque PID W1
  //_m3508.set_v_pid_param(0, 15.4, 0.01, 0.005);  // Velocity PID W1
  //_m3508.set_p_pid_param(0, 9.5, 0.01, 0.3);     // position PID W1
  
  //suggest position mode setting 
  _m3508.set_i_pid_param(0, 1.5, 0.001, 0.000005); // Torque PID W1
  _m3508.set_v_pid_param(0, 5.5, 0.001, 0.005);  // Velocity PID W1
  _m3508.set_p_pid_param(0, 12.5, 0.4, 0.35);     // position PID W1
  
  // set the LP filter of the desire control
  _m3508.profile_velocity_CCW[0] = 12000;//Maximum is 12000 for c620 and 10000 for c610
  _m3508.profile_velocity_CW[0] = -12000;
  _m3508.profile_torque_CCW[0] = 16000; //Maximum is 16000 for c620 and 10000 for c610
  _m3508.profile_torque_CW[0] = -16000;
  // set the current limit, overcurrent may destory the driver
  _m3508.motor_max_current = 16000; // 10000 max for c610+2006 16000 max for c620
  
  // output position = motor rotation(deg) * gear ratio.  2006 = 1:36 , 3508
  // = 1:19 rotate 180 deg
  _m3508.set_position(0, 0);// set starting position as 0
  //_m3508.set_velocity(0, 0);
  while (true) {
    if (!upbutton) {
      state = !state;
    }
    if (state) {
      led = 1;
      t2.start();
      counter++;
      //delay 1 sec
      if (counter >= 1002) {
        counter = 0;
        t2.stop();
        step++;
        t2.reset();
      }
    } else {
      led = 0;
      t2.stop();
      step = 0;
    }
    switch (step) {
    case 1:
      _m3508.set_position(0, 60 * 19);
      //_m3508.set_velocity(0, 1000);
      break;
    case 2:
      _m3508.set_position(0, 120 * 19);
      //_m3508.set_velocity(0, 2000);
      break;
    case 3:
      _m3508.set_position(0, 180 * 19);
      //_m3508.set_velocity(0, 4000);
      break;
    case 4:
      _m3508.set_position(0, 240 * 19);
        //_m3508.set_velocity(0, 6000);
      break;
    case 5:
      _m3508.set_position(0, 300 * 19);
      //_m3508.set_velocity(0, 8000);
      break;
    case 6:
      _m3508.set_position(0, 360 * 19);
      //_m3508.set_velocity(0, 10000);
      break;
    case 7:
      _m3508.set_position(0, 300 * 19);
      //_m3508.set_velocity(0, -10000);
      break;
    case 8:
      _m3508.set_position(0, 240 * 19);
      //_m3508.set_velocity(0, -8000);
      break;
    case 9:
      _m3508.set_position(0, 180 * 19);
      //_m3508.set_velocity(0, -7000);
      break;
    case 10:
      _m3508.set_position(0, 120 * 19);
      //_m3508.set_velocity(0, -6000);
      break;
    case 11:
      _m3508.set_position(0, 60 * 19);
      //_m3508.set_velocity(0, -5000);
      break;
    case 12:
      _m3508.set_position(0, 0);
      //_m3508.set_velocity(0, 0);
      step = 0;
      break;
    default:
      _m3508.set_position(0, 0);
      break;
    }
    // need to run in order to keep the PID running
    _m3508.c620_read();
    _m3508.c620_calc();
    // delay 1 millisec = 1000Hz = c610/c620 can bus feedback frequency
    ThisThread::sleep_for(1ms);
  }
}
```

## RM motor features

### Introduction
RoboMaster Motor(RM motor), is the black cylindrical box as shown in the photo below:
![](https://stormsend1.djicdn.com/tpc/uploads/photos/1928/large_fcfcea55-88e3-4521-a81e-fa7ea136ac0d.jpg)
and we need a motor driver to convert the signal to what the motor "understand" which is called a RM-ESC:
![](https://product1.djicdn.com/uploads/sku/cover/08aa5010-64ec-447d-a568-1d2eb597f345@medium.png)

### Wiring and power supply
![](https://i.imgur.com/HSK66yp.png)
- Both [1] and [2] will attached to the RM motor.
- Connect the [5] to a 24V cells
- [7] is CAN port [8] is PWM port
    - should only choose **1 port** to use

### Potential Damage
1. Collision
    * Dropping
    * Somehow your robot has the RM motor or ESC exposed to external objects and your robot crashed into them
2. Overheating
    * Cause
        1. Overdriving (Forcing the motor to provide excessive amount of torque)
        2. Short circuit
            * 24V power wire is shorted and so the motor is provided with too much current
            * Heating effect (Hopefully you have learnt high school physics)
    * Results
        1. Non Permanant Damage
            * Just some warmth
        2. Permanant Damage
            * Electrical component gets damaged
            * More serious cases: **Sparks** and **Smoke**
##### Note
> Please check the temperature of both ESC and RM motor if something weird happened. E.g. Your motor didn't drive when it should, Irrigating smell

### Prevention
1. Make sure that the ESC is tightly tied to the robot internally
2. Ensure there is nothing that blocks the rotation motion of the RM motor (E.g. Loose screws)
    * Turn off the 24V power supply
    * Remove the obstacle if you can or else find a mech member to deal with it
    
2. Overheating
    - Irritating Smell (Smells like celery)
    - Cool down in whatever way you can (Preferably use compressed air in the lab to cool it down)

**Always turn off 24V power supply first when accidents happen**

### Configuration of RM ID
Steps:
1. Find a hard rod with a small end (We use tweezers most of the time)
2. Poke the SET Button[4] in the upper image to set the CAN ID
    - Number of times to press the SET Button = CAN ID + 1
    - The first press is to tell the RM ESC to enter CAN ID configuration mode
3. Watch the LED blink and make sure the CAN ID is correct
    - ID = LED blink time - 1

### Callibarion of the RM motor

#### Background
RM motors can only be controlled when the ESC knows the condition of the motor.
(E.g. Current and inductance)
Even though the RM motor is a commercialized product, 
it is extremely difficult for every single motors to be identical.
Thus, we need to let the ESC to know the actual condition of the RM motor through callibration.

#### Procedures
1. Again, find a hard rod with a small end
2. Press the SET Button for a long time and it will enter the callibration mode
3. Wait untill the LED blinks green quickly.
4. The motor will start spinning but no worries, just wait untill it stops.
5. Please ensure that there is no obstacle bothering the RM motor spinning motion.

### What does the LED signal mean?
TLDR: Any LED signal except for a green blinking LED is not a good sign.
![](https://i.imgur.com/5lHllL5.png)
![](https://i.imgur.com/OcNbReB.png)
![](https://i.imgur.com/saNpZGB.png)

### Electrical Structure of a RM control system
![](https://i.imgur.com/ROzjtKy.png)

### Common Errors
> Arranged in order of occurence

#### *24V power*


| Symptom                | Meaning                           | Solution                                                                                                           |
| ---------------------- |:--------------------------------- |:------------------------------------------------------------------------------------------------------------------ |
| No LED Light at all    | No input power source             | 1. Check if the wire is plugged in or not <br>  2. Check if the fuse is burnt <br> 3. Ask hardware people for help |
| Solid red LED          | No CAN signal from the RM Motor   | Check the 7-pin wire connected to the RM Motor                                                                    |
| Red LED blinking twice | The 3-phase cable is not connected. | Check the connection between RM-Motor and ESC (3-phase cable)                                                              |

#### *Signal*
> Do these when you are sure it's not the power's problem

*CAN:*


| Possible Problem | Diagnosis                                                                                                                                                                                                                                                                                           | Solution                                        |
|:---------------- |:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |:----------------------------------------------- |
| Wrong CAN ID     | Check Check Check                                                                                                                                                                                                                                                                                   | Reinitialize the correct CAN ID                 |
| Wrong CAN Bus    | See if any motors are not moving the way you want                                                                                                                                                                                                                                                   | Send message to the right bus instead           |
| Wrong CAN Port   | Again, Check Check Check                                                                                                                                                                                                                                                                            | I mean, just don't plug it into the wrong hole |
| Broken CAN wire  | 1. Find a hardware member <br> 2. Check with a [DMM in beep mode](https://cdn.sparkfun.com/assets/learn_tutorials/1/01_Multimeter_Tutorial-09.jpg?__hstc=250566617.4b44870ec4a577029c49e44b73bd3bee.1627603200060.1627603200061.1627603200062.1&__hssc=250566617.1.1627603200063&__hsfp=3390389424) | Just get a new wire                             |

*PWM:*


| Possible Problem                                     | Diagnosis                                                                                                                                                                                                                                                                                           | Solution                            |
|:---------------------------------------------------- |:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |:----------------------------------- |
| Incorrect frequency/ on time initialized in the code | 1. Check with Osciloscope <br> 2. Check your code                                                                                                                                                                                                                                                   | Change it back lol                  |
| False soldering for the pwm pins on the mainboard    | Find a hardware member for help                                                                                                                                                                                                                                                                     | Hardware member should deal with it |
| Broken PWM Wire                                      | 1. Find a hardware member <br> 2. Check with a [DMM in beep mode](https://cdn.sparkfun.com/assets/learn_tutorials/1/01_Multimeter_Tutorial-09.jpg?__hstc=250566617.4b44870ec4a577029c49e44b73bd3bee.1627603200060.1627603200061.1627603200062.1&__hssc=250566617.1.1627603200063&__hsfp=3390389424) | Just get a new wire                 |
| Broken ESC (Basically never happened)                | When you are a **100%** sure you did everything correctly and the motor is not working properly <br>Swap with an ESC that works and see if it's actually faulty                                                                                                                                     | Just get a new ESC                  |




#### CAN vs PWM
Both PWM and CAN can be used to contorl the motor. 
##### PWM:
Advantange:

- Easier to use and implement

Disadvantage:

- PWM can only output signal and can't get the feedback data back. As in real-life condition, the motor cannot be exactly identical (As said from above). If we just output a signal without getting feedback, the speed of motor may not be the same. Therefore, the wheelbase may not move as you want it to be.

##### CAN:
Advantange:

- Can take encoder feedback so we can perform better control for the motors. (Details: [PID_control_method](https://www.youtube.com/watch?v=JFTJ2SS4xyA))

Disadvantage:

- A bit annoying to implement(requires more work)
- Hard to debug when there are problems
    - Could be because of:
        - CAN IC
        - CAN wire
        - Hardware soldering
        - CAN port
        - Code
        - or EVERYTHING


Reference: https://rm-static.djicdn.com/tem/17348/RoboMaster%20C620%20Brushless%20DC%20Motor%20Speed%20Controller%20V1.01.pdf
