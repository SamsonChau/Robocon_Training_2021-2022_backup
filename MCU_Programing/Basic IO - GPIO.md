# Tutorial 2: Basic IO - GPIO

### Table of Content:

1. [Basics](#Basics)
2. [GPIO Configuration](#GPIO-Configuration)
3. [GPIO Input](#Input-GPIO_MODE_INPUT)
4. [GPIO Output](#Writing-GPIO-Output-in-the-Program)
5. [Homework](#Homework)

Mbed OS reference:

https://os.mbed.com/docs/mbed-os/v6.13/mbed-os-api-doxy/group__drivers-public-api-gpio.html

## Basics

### Fundamental Program Structure

```c
#include "mbed.h"

DigitalOut myled(LED1);
DigitalIn userButton(USER_BUTTON);

bool buttonDown = false;

int main()
{
    // check that myled object is initialized and connected to a pin
    if (myled.is_connected()) {
        printf("myled is initialized and connected!\n\r");
    }

    // Blink LED
    while (1) {
         if (userButton) {  // button is pressed
             if  (!buttonDown) {  // a new button press
              led = !led;                 // toogle LED
              buttonDown = true;     // record that the button is now down so we don't count one press lots of times
              wait_ms(10);              // ignore anything for 10ms, a very basic way to de-bounce the button. 
             } 
           } else { // button isn't pressed
            buttonDown = false;
           }
         }
    }
}
```

The HAL_GetTick() function is the main thing to use for timing control in software. It counts up by one every millisecond.

**Some Reminders**: Try not to use `int`, `long`, `long long`, `float`, `double` for embedded system programming (Do you know why?). Instead, use the followings:

| Short form | Full name          | Meaning                  |
| :--------: | :----------------- | :----------------------- |
| `uint8_t`  | unsigned char      | unsigned 8 bits integer  |
|  uint16_t  | unsigned short     | unsigned 16 bits integer |
|  uint32_t  | unsigned long      | unsigned 32 bits integer |
|  uint64_t  | unsigned long long | unsigned 64 bits integer |
|   int8_t   | signed char        | signed 8 bits integer    |
|  int16_t   | signed short       | signed 16 bits integer   |
|  int32_t   | signed long        | signed 32 bits integer   |
|  int64_t   | signed long long   | signed 64 bits integer   |

Both full name and short form work.

**Note:** `float` and `double` should only be used when necessary as floating point calculation normally takes more time :cry:

## What is GPIO?

A **general-purpose input/output (GPIO)** is an digital signal pin on an integrated circuit or electronic circuit board whose behavior — including whether it acts as an input or output — is controllable by the user at run time.

In short, GPIO is for outputting and reading HIGH [1] and LOW [0] values. :smiling_face_with_smiling_eyes_and_hand_covering_mouth:
(Sometimes we will use GPIO for analogue values but don't worry about that for now)

### GPIO Configuration

Every pin on the MCU can be used as GPIO, they are divided into a blocks of 16 pins due to internal structure

Each port is named by letters: `PA`, `PB`, `PC`...

Each port within the pin is numbered from 0: `PA_0`, `PA_1`, `PA_2`...

So every pin is referred to by a pair of values: the port and the pin: `Port A pin3` or when humans are talking we say `PA3`

We will often use `#define` to give more meaningful names to the ports and pins, for ease of use:

```c
#define LED1_Pin             PA_6
#define USER_BUTTON_PIN      PC_13
#define LED3_PIN             PB_6
```

These defines need to follow the board and  the MCU you are using, the every MCUs have their chips set.

Any hardware that you want to use must be first initialized. This is basically setting it up so it can work they way you want it to.

##### Explanations

###### _Input_ `Digital`

- `PULLUP` and `PULLDOWN`- The GPIO Pin can be used to read the logical value of the pin
  - The point is to deal with the secret 3rd state of binary digital signals: **_floating pins_**. What happens when a pin is connected to nothing at all?
  - Noise will cause you to read a mostly random value
  - The pull-up or pull-down gives a "_weak_" connection from the pin to either a high or low voltage. It gives a defined value to a floating pin while being weak enough to be easily overriden by any external signal.

![when the GPIO is connected to a pull up resistor](https://i.imgur.com/iS3Od90.jpg)] ![when the GPIO is connected to a pull down resistor](https://i.imgur.com/sd7FdFC.jpg)

> One of the two set-ups here will read a high voltage when the button is on and low voltage when the button is off, while the other one is the opposite, can you tell me which is which and why?

###### _Output_

- `Open Drain` and `Push pull` - The GPIO Pin can be used to output a digital signal using a pair of switches (see figure below)
  - Push-pull(PP) uses the 2 switches to connect the pin to either high voltage or low voltage, it pushes or pulls the voltage to the level assigned
  - Open-drain(OD) is similar to the above but does not use the upper switch, thus it outputs a low voltage or completely disconnects the pin
  - `both` are the same but are for when the pin output is to be controlled by another bit of hardware, don't worry about it for now

Inside the MCU lives a pair of switches(transistors):<br>
![](https://i.imgur.com/UerwY9k.png)

#### Reading GPIO Input in the Program

The `DigitalRead` function reads the GPIO input.

```c
//Define
DigitalIn userButton(USER_BUTTON);

// usage
uint8_t Button_State = userButton; 
// returns 0 or 1
```

The Type  `DigitalIn` functions returns 1 or 0 of that respective pin of the parameter.

#### Writing GPIO Output in the Program

The following macros can be found in `main.h`

The `gpio_set(gpio)` macro sets the GPIO pin to be 1.

The `gpio_reset(gpio)` macro resets the GPIO pin to 0.

The `gpio_toggle(gpio)` macro toggles the GPIO pin. (i.e. changes the GPIO pin state to 1 if it was originally 0 and vice versa)

#### Examples:

```c
// Turns on the LED1 (logical 1)
LED1 = 1;
//or
LED1.write(1);

// Turns on the LED1 (logical 0)
LED1 = 0;
//or
LED1.write(0);

// Toggles the State of LED1
LED1 = !LED1;
```

### Full Example (Flickering the LED1 for every 500 ticks)

```c
#include "mbed.h"

DigitalOut myled(LED1);

int main()
{
    // check that myled object is initialized and connected to a pin
    if (myled.is_connected()) {
        printf("myled is initialized and connected!\n\r");
    }

    // Blink LED
    while (1) {
        myled = 1;          // set LED1 pin to high
        ThisThread::sleep_for(500);

        myled.write(0);     // set LED1 pin to low
        ThisThread::sleep_for(500);
    }
}
```

### Simplest uses for GPIO

On the board that we have given you there are 4 LEDs,and 2 Buttons

![](https://i.imgur.com/6ekaG6n.png)

![](https://i.imgur.com/M32ZjoS.png)

**_Thinking Time_**:

What should the GPIO Mode and Pull be for each of the above and why?

:heavy_exclamation_mark:Can you spot out the mistakes in the code boilerplate in terms of controlling the led and reading input based on this schematics?

When you want to use a GPIO Pin there are some main steps you should remember:

1. Find which GPIO you want to use.
2. Initialize the GPIO Pin. Remember which mode and pull you need for the pin to do its job
3. Use the GPIO Functions to read or write the digital signals as you want

#### Button Example

```c
// Initialising all gpio
DigitalOut BTN1(PC_13);
// Remember the onboard button must be used with a pull up resistor

uint8_t pressed = Button.read(BTN1);
//or
uint8_t pressed = BTN1;
// Remember the onboard button will give a low signal when clicked, and a high signal otherwise
```

Do classwork [Task 1](https://github.com/HKUST-Robotics-Team/HKUST-Robotics-Team-SW-Tutorial-2021/blob/main/Tutorial%202%20-%20Basic%20IO/Classwork%20%26%20Homework.md#task1) when you see this!

## Homework

#### Edge vs Level Triggering

Consider 2 uses for a button:

Q1 Level Triggering (5 marks)

- While the button is down print `Hello, (Your name)` on Serial console(2m), while it is not, flash the LED (2m). Two actions should not happen simultaneously (1m).
  - In this case every time the loop comes around, we are concerned with the current state (or level) of the buttons GPIO Pin
  - The implementation of the button reading here should be obvious and simple

Q2 Edge Triggering (10 marks)

- What if, we wanted to print `Hello, (Your name)` for 1 second when the button is clicked (<200 ms), so holding the button does nothing more (5m). When the button is released, we want to flash the LED for 1 second.(5m). The process repeats. i.e. it will print text again if you click the button. 
  - The event of a signal going from low to high is called the _rising edge_ and the opposite is the _falling edge_
  - The `gpio_read()` macro gives us the current state, but edge triggering also requires knowledge of the past state as well as some logic

**_Thinking Time_**: How can we design some code that can call a function _only_ when the button is first clicked? (Rising edge)

#### (Bonus) - For those who want some fun (15 marks)

- Create a sprite in the middle of the screen. (Can be in any shape other than simple rectangle) (3m)
- It will move to the left for one CHAR_WIDTH when BTN1 is clicked and released, move to the right for one CHAR_WIDTH when BTN2 is clicked and released. (5m)
- Long press (> 300ms) and release BTN1 for it to move up for one CHAR_HEIGHT. Long press and release BTN2 for it to move down for one CHAR_HEIGHT. (7m)