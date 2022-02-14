# Tutorial 3 - Advanced IO(PWM)

## Concept of PWM

### Why do we need PWM signal?

For controlling the DC motor and servo motor.

The Solution:

**Pulse Width Modulation(PWM)** - We’ll take a look into the total amount of *on-off time* of a signal. And utilize this data, rather than just the current state, to convey useful information.

## How to Generate PWM Signals?

Using Timers.

The above term will be explained in detail in the following notes. So no need to worry.

### There are two main components of PWM generation:

1. The output frequency
2. The on-time (The time of “HIGH voltage”)
   - (***Duty Cycle***) which is **on-time** to **period** ratio
   - in Second

![123](https://i.imgur.com/kcznbWB.png)

The data-sheet will usually provide the frequency and on-time (pulse) to use. The SG90 servo uses 50 Hz and 1-2ms on-time.

We will also use PWM signal to control a DC motor, which duty cycle implies the output power of the DC motor. For example letting the motor to spin at a lower speed.

## The output frequency (in second)

#### General concept frequency:

f=1Periodf=1Period

#### Frequency of clock (MCU_Clock):

The timer of clock (in our borad we are using 84MHz)

#### Frequency of PWM:

The output (or desired frequency) for the motors

#### Prescaler value(PSC):

a 16-bit unsigned integer

#### Auto-reloaded counter(ARR):

a 16-bit unsigned integer

![img](https://imgur.com/YF8xjhF.png)

#### The Prescaler value of the upper picture is 1

As you can see, when there are 2 peaks in the **MCU_Clock**, 1 peak in **Clock after Prescaler** is generated. So the prescaler value must be 2 right? No! As we are programmers, we always count from 0. So, the **Prescaler value = 1.**

#### The Auto-reloaded counter of the upper picture is 36

As you can see, when there are 37 peaks in **Clock after Prescaler**, the **Auto-reloaded counter** increases by 1, when the value is > 36, 1 peak in Counter overflow is generated and the counter is reset to 0. Again, we are programmers, so the **Auto-reloaded counter = 36.**

On the above picture you can see that the **prescaler value** and **auto-reload counter** help reduce the frequency of the **MCU_Clock** and generate a lower frequency.

The **Prescaler value** and **Auto-reload value** are limited to a 16-bit unsigned integer only. Thus, the maximum value of both values is 216−1=65535216−1=65535.

The purpose of having both of them is that our MCU runs at a high frequency. If we were to work with servos (assume they require 50Hz) and use only the Prescaler or only the Auto-Reload, we won’t be able to reduce to the targeted frequency. That’s why we need both.

The difference between the two is that **Prescaler value** is just aiming to reduce the frequency, while **auto-reload counter** also aims as a counter.

After all, how do we get the output frequency???

FreqencyOutput=Frequencyofclock(PrescalerValue+1)⋅(Auto−reloadedcounter+1)FreqencyOutput=Frequencyofclock(PrescalerValue+1)⋅(Auto−reloadedcounter+1)

### Classwork 1

If we need a frequency output of 50Hz, what are the 3 possible combinations of prescaler value and auto-reload value? (Given that the clock frequency is 84MHz)

## The On-time(duty cycle)

#### Duty cycle:

According to the figure below, the number on the left-hand side is the duty cycle, it means the percentage of time that a signal is given as “high”(or 5V). i.e.On−TimePeriodi.e.On−TimePeriod

![img](https://i.imgur.com/Y8cyv7H.png)

By only looking at the picture, you may not understand what is happening. It maybe easier for us - programmers to understand through actual code.

```
#include "mbed.h"

// Adjust pin name to your board specification.
// You can use LED1/LED2/LED3/LED4 if any is connected to PWM capable pin,
// or use any PWM capable pin, and see generated signal on logical analyzer.
PwmOut led(LED2);

int main()
{
    // specify period first, then everything else
    led.period(4.0f);  // 4 second period
    led.write(0.50f);  // 50% duty cycle
    while (1);         // led flashing
}
```

In Mbed OS you can define the pwm pin as ` PwmOut ` the period and the duty cycle can be set easily via the upper command.

DutyCycle=CCR(CompareValue)+1ARR(auto−reloadcounter)+1=On−timePeriodDutyCycle=CCR(CompareValue)+1ARR(auto−reloadcounter)+1=On−timePeriod

### How to choose the value of auto-reload counter and prescaler value

From the formula:

FreqencyOutput=Frequencyofclock(PrescalerValue+1)⋅(Auto−reloadedcounter+1)FreqencyOutput=Frequencyofclock(PrescalerValue+1)⋅(Auto−reloadedcounter+1)

There are many combinations of prescaler value and auto-reload counter that can generate the same frequency output. How can we choose a better value?

As you may notice, the ARR acts as an denominator in the **Duty Cycle formula**, therefore if we want to output a short **on-time**, we need to have larger denominator. Notice that both of the CCR and ARR has to be an 16-bit unsign integer.

As a result, **Larger Auto-reload Value and Smaller Prescaler value would be better when outputing a short on-time.**

### Classwork 2

If we need a frequency output of **50Hz** and on-time of **0.5ms**, what are the possible combinations of prescaler value, auto-reload value and compare value? (Given that the clock frequency is 84MHz)

