---
hidden: true
title: "85 Customizable UART String"
weight: 86
---

## 85 : Customizable UART String

* Author: Tiny Tapeout 02 (J. Rosenthal)
* Description: This design Supports sending multiple ASCII characters over UART.
* [GitHub repository](https://github.com/psychogenic/tt03-UARTstring)
* [Most recent GDS build](https://github.com/psychogenic/tt03-UARTstring/actions/runs/4773250588)
* [Wokwi](https://wokwi.com/projects/347144898258928211) project
* [Extra docs](https://wokwi.com/projects/347144898258928211)
* Clock: 300 Hz
* External hardware: Arduino, computer with serial monitor connected to the Arduino



### How it works

This circuit implements five shift registers with 21 bits: seven idle bits, one start bit, eight data bits, one stop bit, and four more idle bits. The circuit supports transmitting a string of ASCII characters 

### How to test

Connect an Arduino serial RX pin to the eight output pin (Output[7]). In the Arduino code, set the serial baud rate Serial.begin(<baud rate>); in the *.ino file to 300. Set the PCB clock frequency to 300 Hz as well. Set the slide switch to the clock. Set SW7 to OFF ('Load'). Set SW8 to ON ('Output Enable'). Set SW7 to ON ('TX').

### IO

| # | Input        | Output       |
|---|--------------|--------------|
| 0 | clock  | segment a (Output Enable) |
| 1 | N/A  | segment b (Load/TX) |
| 2 | N/A  | segment c |
| 3 | N/A  | segment d |
| 4 | N/A  | segment e |
| 5 | N/A  | segment f |
| 6 | Load/TX  | segment g |
| 7 | Output Enable  | UART Serial Out |
