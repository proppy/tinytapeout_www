---
hidden: true
title: "204 Configurable Gray Code Counter"
weight: 205
---

## 204 : Configurable Gray Code Counter

* Author: Eric Swalens
* Description: A configurable counter driven by 2-channel Gray code
* [GitHub repository](https://github.com/swalense/tt02-graycode_counter)
* [Most recent GDS build](https://github.com/swalense/tt02-graycode_counter/actions/runs/3602237769)
* HDL project
* [Extra docs](https://github.com/swalense/tt02-graycode_counter#readme)
* Clock: 5000 Hz
* External hardware: A source of Gray code; filtering and Schmitt triggers may be required if a mechanical encoder is used.




### How it works

The module is an 8-bit configurable counter modified by Gray code (aka 2-bit quadrature code);
it aims at easing the integration of incremental rotary encoders into projects.
The counter value is given as a (truncated to 5 bits) parallel or (8 bits, no parity, 1 stop bit) serial UART output.
Other outputs include the "direction" of progression of the Gray code, and a PWM signal for which the duty cycle is proportional to the counter value.

Some basic (optional) debouncing logic is included; any pulse inverting the direction must be followed by a second pulse in the same direction
before the change is registered.

Additional features include support for wrapping (the counter rolls over at the minimum and maximum value),
and a "gearbox" that selects the X1 (1 pulse per 4 transitions), X2 (2 pulses) or X4 (4 pulses) output of the Gray code decoder driving the counter
depending on the speed at which the channels change; this can provide some form of "acceleration".
The initial and maximum values of the counter can also be set.  

Encoders with twice the number of detents compared to the number of pulses per round (e.g. 24 detents / 12 PPR) are supported 
by setting the input "update on X2" high or forcing it with the configuration parameter.

After reset the module is configured as a basic 5-bit counter which can then be further modified by sending a 32-bit word over the SPI interface.
This word sets the following options (reset value between parentheses):

- gearbox enable (0)

- debounce logic enable (1)

- wrap enable (0)

- Gray code value for X1 (0)

- force update on X2 (0), this overrides a low value at the input pin (the value for X1 selects which transitions are taken into consideration)

- gearbox timer value (n/a, gearbox is disabled)

- counter initial value (0)

- counter maximum value (31)

See link to GitHub for possible errata.


### How to test

For a basic test connect a device generating Gray code and retrieve the counter value at the parallel or serial outputs with a microcontroller or other circuitry.

To further configure the module send some configuration word over the SPI interface (mode 0, MSB first, CS is active low).
The 32-bit configuration word is constructed a follows (bits between brackets):

- [24:31] maximum counter value

- [16:23] initial counter value after configuration

- [8:15] gearbox timer

- [6:7] unused

- [5:5] force update on X2

- [3:4] X1 value

- [2:2] debounce enable

- [1:1] wrap enable

- [0:0] gearbox enable

The gearbox is implemented with a 5-bit threshold value; it is incremented by the X4 output of the decoder and decremented by a timer
(this threshold is then divided by 8 to select the gear, giving 0: X1, 1: X1, 2/3: X4).
Therefore the result depends on the clock frequency and the speed at which the Gray code transitions. The gearbox timer is exposed to enable tuning
the interval between two updates by the timer.
For a rotary encoder with detents one can suggest using *clock_hz / (detents x transitions - 16)* as a starting point to determine a suitable value,
where detents is the number per turn (e.g. 24) and transitions is the number per detent (e.g. 4). That is, 62 for a common 24 detents / 24 PPR encoder.

The 8-N-1 UART serial output shifts 1 bit out at each clock cycle. The receiving serial port therefore needs to be configured at the same speed as the clock.

The PWM frequency is derived from the maximum counter value. It might be unsuitable for visual feedback, e.g. driving a LED, for large values with a low
clock frequency as the LED will appear blinking.


### IO

| # | Input        | Output       |
|---|--------------|--------------|
| 0 | clock  | UART serial output |
| 1 | reset  | PWM signal |
| 2 | channel A  | direction |
| 3 | channel B  | counter bit 0 |
| 4 | update on X2  | counter bit 1 |
| 5 | SPI CS  | counter bit 2 |
| 6 | SPI SCK  | counter bit 3 |
| 7 | SPI SDI  | counter bit 4 |
