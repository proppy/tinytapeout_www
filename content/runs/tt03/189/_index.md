---
hidden: true
title: "189 Sigma-Delta ADC/DAC"
weight: 190
---

## 189 : Sigma-Delta ADC/DAC

* Author: Adam Greig
* Description: Simple ADC and DAC
* [GitHub repository](https://github.com/adamgreig/tt02-adc-dac)
* [Most recent GDS build](https://github.com/adamgreig/tt02-adc-dac/actions/runs/3598338001)
* HDL project
* [Extra docs](https://github.com/adamgreig/tt02-adc-dac)
* Clock: 6000 Hz
* External hardware: Comparator, resistor, capacitor



### How it works

This project is built on a simple sigma-delta DAC. The DAC is given an n-bit
control word and generates a single-bit digital output where the pulse
density is proportional to that control word. By integrating this pulse
train, for example with an RC filter, an analogue output voltage is produced.

The ADC operates by generating an analogue output voltage which is compared
to the analogue input by an off-chip comparator. The comparator result is
used as a digital input to a simple control loop that adjusts the output
voltage so that it tracks the input signal. The control word for the DAC
generating the output voltage is then the ADC reading. This control word
is regularly transmitted as hex-encoded ASCII over a UART running at the
clock rate.

A second dedicated 8-bit DAC is controlled by received words over a UART.
Transmit the control word at 1/10th the clock speed into `uart_in`, and
add a second external RC circuit to filter `dac_out` to an analogue voltage.


### How to test

Ensure in[0] is clocked. Connect out[0] through a series resistor to both
a capacitor to ground and the non-inverting input of a comparator. Connect
the analogue input to measure to the inverting input, and connect the
comparator output to in[2]. Connect out[1] to a UART receiver at the clock
rate and receive ADC readings as hex-encoded ASCII lines.

Connect out[2] to a second RC filter, and feed one-byte DAC settings
to the UART on in[3] at a baud rate 1/10th the clock. Measure the
resulting analogue output.


### IO

| # | Input        | Output       |
|---|--------------|--------------|
| 0 | clock  | adc_out |
| 1 | reset  | uart_out |
| 2 | adc_in  | dac_out |
| 3 | uart_in  | none |
| 4 | none  | none |
| 5 | none  | none |
| 6 | none  | none |
| 7 | none  | none |
