# Lesson Four - Communication Protocol 

## Introduction

Imagine you want to transfer 8-bit data to another board, one way is to connect 8 wires to another board: ![img](https://cdn.sparkfun.com/r/700-700/assets/c/a/c/3/a/50e1cca6ce395fbc27000000.png)

However, this is terrible. What if we want to transfer 32-bit data? We would need a lot of wires.

This calls for communication protocols, which aims to allow devices to communicate with each other using as few wires as possible.

![img](https://cdn.sparkfun.com/r/700-700/assets/e/5/4/2/a/50e1ccf1ce395f962b000000.png)

## Universal Asynchronous Receiver-Transmitter (UART)

UART is a protocol used for serial communication and is used when the rate of transmission is not a concern. For example, UART could be used to send control commands but it isn’t suitable to send high resolution images.

In our embedded system, UART is commonly used between two different boards or when trying to send data from an mobile/desktop application to your board using bluetooth.

### Sending and Receiving Data with UART

UART sends your data in serial, **asynchronous** mode. Serial communication is the process of sending data one bit at a time, and asynchronous means that there is no common clock signal between the sender and receivers.

In other words, the clocks of the STM32 and the bluetooth device may be different. This is why you need to set the baud rate on both sides.

![img](https://www.circuitbasics.com/wp-content/uploads/2016/01/Introduction-to-UART-Basic-Connection-Diagram-768x376.png)

For example, if we want to transmit data from UART1 to UART2, data will be sent from the **TX pins** and received by the **RX pins**.

### Configuring UART Ports

![](https://os.mbed.com/media/uploads/adustm/nucleo_f446re_morpho_left_2016_7_22.png)

![](https://os.mbed.com/media/uploads/adustm/nucleo_f446re_morpho_left_2016_7_22.png).

**1. Baud rate**:

<details class="part" data-startline="41" data-endline="51" data-position="2166" data-size="1493" style="box-sizing: border-box; --tw-border-opacity: 1; border-color: rgba(231, 231, 231, var(--tw-border-opacity)); --tw-shadow: 0 0 #0000; --tw-ring-inset: var(--tw-empty, ); --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgba(59, 130, 246, 0.5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; display: block;"><summary style="box-sizing: border-box; --tw-border-opacity: 1; border-color: rgba(231, 231, 231, var(--tw-border-opacity)); --tw-shadow: 0 0 #0000; --tw-ring-inset: var(--tw-empty, ); --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgba(59, 130, 246, 0.5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; display: list-item; cursor: pointer;">What is baud rate?</summary></details>



**Possible baud rate values**: Baud rates have predefined standards (e.g. 9600, 14400, 19200, 38400, 57600, 115200, 128000 and 256000).

`115200` is recommended in this tutorial. **Make sure the baud rates on both sides of the communication are the same.**

**2. Word Length, Parity and Start/Stop Bits**:

During transformation, data is sent in **a packet**, and the size of data frame is 5 - 9 bits.

![img](https://www.circuitbasics.com/wp-content/uploads/2016/01/Introduction-to-UART-Packet-Frame-and-Bits-2.png)

You do not need to modify these parameters for now, just use the default values and make sure the config on both sides of communication is the same.

### Initialization of UART Pins

Like most board/pin controls you have learnt so far, a UART port needs to be initialised before it is available for use. 

Serial channels have the following characteristics:

- TX and RX pins - you can specify either pin as Not Connected (NC) for simplex (unidirectional) communication or both as valid pins for full duplex (bidirectional) communication.
- Baud rate - predefined speed at which data is sent and received on the UART interface. Standard baud rates include 9600, 19200 and 115200.

Data is transmitted using packets of configurable sizes divided in different sections, which include:

- Start bit: indicates the start of UART data transmission.
- Data frame: can be 5 to 8 (or 9 if a parity bit is not used) bits for the actual data being transferred.
- Parity bit: optional bit, used for data error detection.
- Stop bits: 1-2 bits to signal the end of a data packet.



## Configuration

The following parameters can be configured at object instantiation:

- *TX*.
- *RX*.
- *Baud rate*.

The default baud rate value is configured in `mbed-os/platform/mbed_lib.json`.

The following parameters can be configured after an `BufferedSerial` object instantiation.

- *Baud rate*.
- *Data frame length*.
- *Parity bit*.
- *Stop bits*.

The default settings for a microcontroller are set as *9600-8-N-1*, a common notation for serial port settings. This denotes a baud rate of 9,600 (9600), eight data frame length (8), no parity bit (N) and one stop bit (1).

You can also configure hardware flow control if necessary.

You can view more information about the configurable settings and functions in the class reference.Transmitting and Receving Data

After the initialization, you can start sending and receiving data by calling functions from the HAL Library.

Helper functions documentation for your reference:https://os.mbed.com/docs/mbed-os/v6.15/apis/serial-uart-apis.html

**Example:**

```
#include "mbed.h"

// Maximum number of element the application buffer can contain
#define MAXIMUM_BUFFER_SIZE                                                  32

// Create a DigitalOutput object to toggle an LED whenever data is received.
static DigitalOut led(LED1);

// Create a BufferedSerial object with a default baud rate.
static BufferedSerial serial_port(USBTX, USBRX);

int main(void)
{
    // Set desired properties (9600-8-N-1).
    serial_port.set_baud(9600);
    serial_port.set_format(
        /* bits */ 8,
        /* parity */ BufferedSerial::None,
        /* stop bit */ 1
    );

    // Application buffer to receive the data
    char buf[MAXIMUM_BUFFER_SIZE] = {0};

    while (1) {
        if (uint32_t num = serial_port.read(buf, sizeof(buf))) {
            // Toggle the LED.
            led = !led;

            // Echo the input back to the terminal.
            serial_port.write(buf, num);
        }
    }
}


```

### Sending Data from your STM32 to computer

> Coolterm: https://www.macupdate.com/app/mac/31352/coolterm HTerm (Windows only): http://der-hammer.info/pages/terminal.html TeraTerm: https://ttssh2.osdn.jp/index.html.en

We use Coolterm for demonstration, but there are different ways to interact with UART signals. Some examples are Tera term, mobile applications, and python/C# programs written by yourself using Bluetooth stack. However, we may not be able to provide support to people not using Coolterm.

### Connecting STM32 to the computer via USB-TTL 

Locate the UART1/UART3 port on your STM32 (it’s below your ST-Link port/flashing port) and connect your TTL to the UART port as follows.

| TTL   | Uart Port |
| :---- | :-------- |
| `5V0` | `V`       |
| `TXD` | `R`       |
| `RXD` | `T`       |
| `GND` | `G`       |

If you are using HC-05 (one of the bluetooth modules), please follow the pin arrangement below:

#### Main board side

| Bluetooth | Uart Port |
| :-------- | :-------- |
| `5V`      | `V`       |
| `TX`      | `R`       |
| `RX`      | `T`       |
| `GND`     | `G`       |



### Setting up and using CoolTerm

Open coolterm and set up as below:

```
**option -> serial port-> port = port you used to connect your TTL to your computer
serial port -> baud rate = 115200
serial port -> data bits = 8 bit
serial port -> parity = none
serial port -> stop bits = 1**

**terminal -> terminal mode = line/raw mode 
terminal -> Enter key emulation = CR/LF
terminal -> local echo = check
Press connect and run your board, and the messages should appears.**
```



## Classwork #1: Send your name and student from MCU to coolterm.

What you should see in coolterm: ![img](https://i.imgur.com/y152z6j.png)

## Classwork #2: Send a string to MCU, then echo it back to coolterm

![img](https://i.imgur.com/xAm0BFS.png)

## Classwork #3: Control LEDs by sending commands from computer to MCU through bluetooth.

Strings to send:

- Turn on LEDx: `+x`

- Turn off LEDx: 

  ```
  -x
  ```

  - e.g. to turn on LED1: `+1`; to turn off LED3: `-3`.

- Toggle LEDx: 

  ```
  /x
  ```

  - LED should keep toggling until a `+` or `-` command is sent.

You should parse these commands on the mainboard and perform the appropriate action.



 