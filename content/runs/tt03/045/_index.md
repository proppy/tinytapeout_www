---
hidden: true
title: "45 MicroTapeout (of sky130 cells)"
weight: 46
---

## 45 : MicroTapeout (of sky130 cells)

* Author: htfab
* Description: 395 standard cells with a mux to select between them
* [GitHub repository](https://github.com/htfab/microtapeout)
* [Most recent GDS build](https://github.com/htfab/microtapeout/actions/runs/4782840402)
* HDL project
* [Extra docs]()
* Clock: 10000 Hz
* External hardware: 

![picture](images/layout-medium.jpg)

### How it works

Digital chip designs are usually written in a hardware description language like RTL Verilog and then synthesized into a
set of mask layers suitable for fabrication. In order to make both synthesis and verification robust for huge designs,
a modular approach is used where the functionality of the circuit is decomposed into pre-built blocks called _standard cells_
with well-known and thoroughly tested behaviour and layout.

This design contains a copy of most standard cells in the
[sky130\_fd\_sc\_hd](https://antmicro-skywater-pdk-docs.readthedocs.io/en/test-submodules-in-rtd/contents/libraries/sky130_fd_sc_hd/README.html)
library along with a multiplexing mechanism that allows exposing any of them to the input/output pins.

An MPW shuttle fabricates multiple designs on the same wafer. TinyTapeout merges several projects in a single
shuttle submission. MicroTapeout pushes the limit with each block containing just a single cell.
Apart from the geek factor the fabricated chip can be used by low-level digital design engineers to better
understand the behaviour of the individual standard cells and might even provide some timing insights.

There are 437 standard cells in our library, of which 42 don't produce output or require special power handling.
This leaves us with 395 cells. Each cell has up to 6 inputs and up to 2 outputs for a total of 427 outputs.
The same 6 inputs are fed into each cell in parallel while the 427 outputs are divided into 54 pages of 8 outputs each
with a multiplexer deciding which page is mapped to the output pins.

In order to drive the 6 cell inputs and the 6 bits of input to the mux from a total of 8 input pins we use some
registered logic. Input pin 0 is a clock signal while input pin 1 selects _page mode_. On each rising clock edge
we save input pins 2 to 7 into a page register if page mode is on and into an input register if page mode is off.
Cell inputs are then supplied from the input register and the mux operates on the page register.

Mapping of outputs to pages:

| page   | pin | pin 0/4           | pin 1/5           | pin 2/6           | pin 3/7           |
|--------|-----|-------------------|-------------------|-------------------|-------------------|
| 000000 | 0-3 | conb\_1.h         | conb\_1.l         | buf\_1            | buf\_2            |
|        | 4-7 | buf\_4            | buf\_6            | buf\_8            | buf\_12           |
| 000001 | 0-3 | buf\_16           | bufbuf\_8         | bufbuf\_16        | inv\_1            |
|        | 4-7 | inv\_2            | inv\_4            | inv\_6            | inv\_8            |
| 000010 | 0-3 | inv\_12           | inv\_16           | bufinv\_8         | bufinv\_16        |
|        | 4-7 | and2\_0           | and2\_1           | and2\_2           | and2\_4           |
| 000011 | 0-3 | and2b\_1          | and2b\_2          | and2b\_4          | and3\_1           |
|        | 4-7 | and3\_2           | and3\_4           | and3b\_1          | and3b\_2          |
| 000100 | 0-3 | and3b\_4          | and4\_1           | and4\_2           | and4\_4           |
|        | 4-7 | and4b\_1          | and4b\_2          | and4b\_4          | and4bb\_1         |
| 000101 | 0-3 | and4bb\_2         | and4bb\_4         | nand2\_1          | nand2\_2          |
|        | 4-7 | nand2\_4          | nand2\_8          | nand2b\_1         | nand2b\_2         |
| 000110 | 0-3 | nand2b\_4         | nand3\_1          | nand3\_2          | nand3\_4          |
|        | 4-7 | nand3b\_1         | nand3b\_2         | nand3b\_4         | nand4\_1          |
| 000111 | 0-3 | nand4\_2          | nand4\_4          | nand4b\_1         | nand4b\_2         |
|        | 4-7 | nand4b\_4         | nand4bb\_1        | nand4bb\_2        | nand4bb\_4        |
| 001000 | 0-3 | or2\_0            | or2\_1            | or2\_2            | or2\_4            |
|        | 4-7 | or2b\_1           | or2b\_2           | or2b\_4           | or3\_1            |
| 001001 | 0-3 | or3\_2            | or3\_4            | or3b\_1           | or3b\_2           |
|        | 4-7 | or3b\_4           | or4\_1            | or4\_2            | or4\_4            |
| 001010 | 0-3 | or4b\_1           | or4b\_2           | or4b\_4           | or4bb\_1          |
|        | 4-7 | or4bb\_2          | or4bb\_4          | nor2\_1           | nor2\_2           |
| 001011 | 0-3 | nor2\_4           | nor2\_8           | nor2b\_1          | nor2b\_2          |
|        | 4-7 | nor2b\_4          | nor3\_1           | nor3\_2           | nor3\_4           |
| 001100 | 0-3 | nor3b\_1          | nor3b\_2          | nor3b\_4          | nor4\_1           |
|        | 4-7 | nor4\_2           | nor4\_4           | nor4b\_1          | nor4b\_2          |
| 001101 | 0-3 | nor4b\_4          | nor4bb\_1         | nor4bb\_2         | nor4bb\_4         |
|        | 4-7 | xor2\_1           | xor2\_2           | xor2\_4           | xor3\_1           |
| 001110 | 0-3 | xor3\_2           | xor3\_4           | xnor2\_1          | xnor2\_2          |
|        | 4-7 | xnor2\_4          | xnor3\_1          | xnor3\_2          | xnor3\_4          |
| 001111 | 0-3 | a2111o\_1         | a2111o\_2         | a2111o\_4         | a2111oi\_0        |
|        | 4-7 | a2111oi\_1        | a2111oi\_2        | a2111oi\_4        | a211o\_1          |
| 010000 | 0-3 | a211o\_2          | a211o\_4          | a211oi\_1         | a211oi\_2         |
|        | 4-7 | a211oi\_4         | a21bo\_1          | a21bo\_2          | a21bo\_4          |
| 010001 | 0-3 | a21boi\_0         | a21boi\_1         | a21boi\_2         | a21boi\_4         |
|        | 4-7 | a21o\_1           | a21o\_2           | a21o\_4           | a21oi\_1          |
| 010010 | 0-3 | a21oi\_2          | a21oi\_4          | a221o\_1          | a221o\_2          |
|        | 4-7 | a221o\_4          | a221oi\_1         | a221oi\_2         | a221oi\_4         |
| 010011 | 0-3 | a222oi\_1         | a22o\_1           | a22o\_2           | a22o\_4           |
|        | 4-7 | a22oi\_1          | a22oi\_2          | a22oi\_4          | a2bb2o\_1         |
| 010100 | 0-3 | a2bb2o\_2         | a2bb2o\_4         | a2bb2oi\_1        | a2bb2oi\_2        |
|        | 4-7 | a2bb2oi\_4        | a311o\_1          | a311o\_2          | a311o\_4          |
| 010101 | 0-3 | a311oi\_1         | a311oi\_2         | a311oi\_4         | a31o\_1           |
|        | 4-7 | a31o\_2           | a31o\_4           | a31oi\_1          | a31oi\_2          |
| 010110 | 0-3 | a31oi\_4          | a32o\_1           | a32o\_2           | a32o\_4           |
|        | 4-7 | a32oi\_1          | a32oi\_2          | a32oi\_4          | a41o\_1           |
| 010111 | 0-3 | a41o\_2           | a41o\_4           | a41oi\_1          | a41oi\_2          |
|        | 4-7 | a41oi\_4          | o2111a\_1         | o2111a\_2         | o2111a\_4         |
| 011000 | 0-3 | o2111ai\_1        | o2111ai\_2        | o2111ai\_4        | o211a\_1          |
|        | 4-7 | o211a\_2          | o211a\_4          | o211ai\_1         | o211ai\_2         |
| 011001 | 0-3 | o211ai\_4         | o21a\_1           | o21a\_2           | o21a\_4           |
|        | 4-7 | o21ai\_0          | o21ai\_1          | o21ai\_2          | o21ai\_4          |
| 011010 | 0-3 | o21ba\_1          | o21ba\_2          | o21ba\_4          | o21bai\_1         |
|        | 4-7 | o21bai\_2         | o21bai\_4         | o221a\_1          | o221a\_2          |
| 011011 | 0-3 | o221a\_4          | o221ai\_1         | o221ai\_2         | o221ai\_4         |
|        | 4-7 | o22a\_1           | o22a\_2           | o22a\_4           | o22ai\_1          |
| 011100 | 0-3 | o22ai\_2          | o22ai\_4          | o2bb2a\_1         | o2bb2a\_2         |
|        | 4-7 | o2bb2a\_4         | o2bb2ai\_1        | o2bb2ai\_2        | o2bb2ai\_4        |
| 011101 | 0-3 | o311a\_1          | o311a\_2          | o311a\_4          | o311ai\_0         |
|        | 4-7 | o311ai\_1         | o311ai\_2         | o311ai\_4         | o31a\_1           |
| 011110 | 0-3 | o31a\_2           | o31a\_4           | o31ai\_1          | o31ai\_2          |
|        | 4-7 | o31ai\_4          | o32a\_1           | o32a\_2           | o32a\_4           |
| 011111 | 0-3 | o32ai\_1          | o32ai\_2          | o32ai\_4          | o41a\_1           |
|        | 4-7 | o41a\_2           | o41a\_4           | o41ai\_1          | o41ai\_2          |
| 100000 | 0-3 | o41ai\_4          | maj3\_1           | maj3\_2           | maj3\_4           |
|        | 4-7 | mux2\_1           | mux2\_2           | mux2\_4           | mux2\_8           |
| 100001 | 0-3 | mux2i\_1          | mux2i\_2          | mux2i\_4          | mux4\_1           |
|        | 4-7 | mux4\_2           | mux4\_4           | ha\_1.c           | ha\_1.s           |
| 100010 | 0-3 | ha\_2.c           | ha\_2.s           | ha\_4.c           | ha\_4.s           |
|        | 4-7 | fa\_1.c           | fa\_1.s           | fa\_2.c           | fa\_2.s           |
| 100011 | 0-3 | fa\_4.c           | fa\_4.s           | fah\_1.c          | fah\_1.s          |
|        | 4-7 | fahcin\_1.c       | fahcin\_1.s       | fahcon\_1.c       | fahcon\_1.s       |
| 100100 | 0-3 | dlxtp\_1          | dlxbp\_1.q        | dlxbp\_1.n        | dlxtn\_1          |
|        | 4-7 | dlxtn\_2          | dlxtn\_4          | dlxbn\_1.q        | dlxbn\_1.n        |
| 100101 | 0-3 | dlxbn\_2.q        | dlxbn\_2.n        | dlrtp\_1          | dlrtp\_2          |
|        | 4-7 | dlrtp\_4          | dlrbp\_1.q        | dlrbp\_1.n        | dlrbp\_2.q        |
| 100110 | 0-3 | dlrbp\_2.n        | dlrtn\_1          | dlrtn\_2          | dlrtn\_4          |
|        | 4-7 | dlrbn\_1.q        | dlrbn\_1.n        | dlrbn\_2.q        | dlrbn\_2.n        |
| 100111 | 0-3 | dfxtp\_1          | dfxtp\_2          | dfxtp\_4          | dfxbp\_1.q        |
|        | 4-7 | dfxbp\_1.n        | dfxbp\_2.q        | dfxbp\_2.n        | dfrtp\_1          |
| 101000 | 0-3 | dfrtp\_2          | dfrtp\_4          | dfrbp\_1.q        | dfrbp\_1.n        |
|        | 4-7 | dfrbp\_2.q        | dfrbp\_2.n        | dfrtn\_1          | dfstp\_1          |
| 101001 | 0-3 | dfstp\_2          | dfstp\_4          | dfsbp\_1.q        | dfsbp\_1.n        |
|        | 4-7 | dfsbp\_2.q        | dfsbp\_2.n        | dfbbp\_1.q        | dfbbp\_1.n        |
| 101010 | 0-3 | dfbbn\_1.q        | dfbbn\_1.n        | dfbbn\_2.q        | dfbbn\_2.n        |
|        | 4-7 | edfxtp\_1         | edfxbp\_1.q       | edfxbp\_1.n       | sdfxtp\_1         |
| 101011 | 0-3 | sdfxtp\_2         | sdfxtp\_4         | sdfxbp\_1.q       | sdfxbp\_1.n       |
|        | 4-7 | sdfxbp\_2.q       | sdfxbp\_2.n       | sdfrtp\_1         | sdfrtp\_2         |
| 101100 | 0-3 | sdfrtp\_4         | sdfrbp\_1.q       | sdfrbp\_1.n       | sdfrbp\_2.q       |
|        | 4-7 | sdfrbp\_2.n       | sdfrtn\_1         | sdfstp\_1         | sdfstp\_2         |
| 101101 | 0-3 | sdfstp\_4         | sdfsbp\_1.q       | sdfsbp\_1.n       | sdfsbp\_2.q       |
|        | 4-7 | sdfsbp\_2.n       | sdfbbp\_1.q       | sdfbbp\_1.n       | sdfbbn\_1.q       |
| 101110 | 0-3 | sdfbbn\_1.n       | sdfbbn\_2.q       | sdfbbn\_2.n       | sedfxtp\_1        |
|        | 4-7 | sedfxtp\_2        | sedfxtp\_4        | sedfxbp\_1.q      | sedfxbp\_1.n      |
| 101111 | 0-3 | sedfxbp\_2.q      | sedfxbp\_2.n      | ebufn\_1/\_2      | ebufn\_4/\_8      |
|        | 4-7 | einvp\_1/n\_0     | einvp\_1/n\_1     | einvp\_2/n\_2     | einvp\_4/n\_4     |
| 110000 | 0-3 | einvp\_8/n\_8     | dg~sd1\_1         | dg~4sd2\_1        | dg~4sd3\_1        |
|        | 4-7 | dm~6s2s\_1        | dm~6s4s\_1        | dm~6s6s\_1        | clkbuf\_1         |
| 110001 | 0-3 | clkbuf\_2         | clkbuf\_4         | clkbuf\_8         | clkbuf\_16        |
|        | 4-7 | clkinv\_1         | clkinv\_2         | clkinv\_4         | clkinv\_8         |
| 110010 | 0-3 | clkinv\_16        | clkinvlp\_2       | clkinvlp\_4       | cdb~4s15\_1       |
|        | 4-7 | cdb~4s15\_2       | cdb~4s18\_1       | cdb~4s18\_2       | cdb~4s25\_1       |
| 110011 | 0-3 | cbd~4s25\_2       | cdb~4s50\_1       | cdb~4s50\_2       | dlclkp\_1         |
|        | 4-7 | dlclkp\_2         | dlclkp\_4         | sdlclkp\_1        | sdlclkp\_2        |
| 110100 | 0-3 | sdlclkp\_4        | lpfii~0p\_1       | lpfii~0n\_1       | lpfii~1p\_1       |
|        | 4-7 | lpfii~1n\_1       | lpfii~latch\_1    | lpfibs~\_1        | lpfibs~\_2        |
| 110101 | 0-3 | lpfibs~\_4        | lpfibs~\_8        | lpfibs~\_16       |                   |

where dg~ = dlygate, dm~ = dlymetal, cdb~ = clkdlybuf, lpfii~ = lpflow\_inputiso, lpfibs~ = lpflow\_isobufsrc.

The design also contains an experimental timing circuit for measuring the switching times of the individual
standard cells using a ring oscillator. This is complicated by (1) a ring oscillator built from standard cells
being necessarily slower than the time to be measured, and (2) several buffering and multiplexing cells plus
wires also included in the measurement.

To offset (1), we can repeat a measurement several times and average them. Since the ring oscillator is not
synchronized to the rest of the chip, this should result in higher timing resolution. To combat (2), we can
compare the results to gate-level simulations and finetune the models until the results match up.


### How to test

Set pin 1 high to switch to page mode. Find the standard cell you would like to test in the table above
and set pins 2-7 to the 6 bit binary page number indicated in the first column. If pin 0 is not connected
to the clock, manually toggle it low and then high to force a clock cycle.
Set pin 1 low to switch to input mode. Set pins 2 and up to the values that should be supplied
to the selected standard cell's input pins. Once again, you may need to manually trigger a clock cycle.
The result should appear on the output pin corresponding to the table column.

To use the experimental timing circuit, make sure pin 0 is in manual mode (not connected to a clock).
First set the page number the same way as above. While still in page mode, set pins 2-7 to the virtual
page number 111pqr where pqr is the cell index within the page. Trigger another clock cycle using pin 0.
Now set pin 1 low to switch to input mode. Set pins 2 and up to the _initial_ cell input values and
toggle pin 0 low and high again. Set pins 2 and up to the _modified_ cell input values and toggle
pin 0 low and high once more. This will latch the standard cell's previous output (i.e. the one for
the _initial_ input) and will connect the ring oscillator to a counter while the output is the same
as the latched value. The counter is connected to output pins 0-5. If the cell output for the
_initial_ and _modified_ inputs are different, this should settle to a value based on the cell
switching time. Otherwise it will keep running indefinitely. Pin 6 is connected to the same gated
clock as the counter but through a massive clock divider, resulting in visible blinking if the counter
is still running. The blinking speed can also be measured to calculate the frequency of the ring
oscillator (which depends on temperature, voltage and process parameters). Pin 7 shows the latched
cell output to help debugging.


### IO

| # | Input        | Output       |
|---|--------------|--------------|
| 0 | clock  | output[8*page+0] / counter[0] |
| 1 | page mode  | output[8*page+1] / counter[1] |
| 2 | input[0] / page[0] / cell[0]  | output[8*page+2] / counter[2] |
| 3 | input[1] / page[1] / cell[1]  | output[8*page+3] / counter[3] |
| 4 | input[2] / page[2] / cell[2]  | output[8*page+4] / counter[4] |
| 5 | input[3] / page[3] / 1 (timing)  | output[8*page+5] / counter[5] |
| 6 | input[4] / page[4] / 1 (timing)  | output[8*page+6] / strobe |
| 7 | input[5] / page[5] / 1 (timing)  | output[8*page+7] / latched value |
