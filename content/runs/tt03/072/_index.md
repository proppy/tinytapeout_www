---
hidden: true
title: "72 RTL Locked QTCore-A1"
weight: 73
---

## 72 : RTL Locked QTCore-A1

* Author: Luca Collini and Hammond Pearce
* Description: A RTL locked accumulator-based 8-bit microarchitecture designed via GPT-4 conversations.
* [GitHub repository](https://github.com/Lucaz97/tt03-RTLlocked-verilog-qtcoreA1)
* [Most recent GDS build](https://github.com/Lucaz97/tt03-RTLlocked-verilog-qtcoreA1/actions/runs/4780876099)
* HDL project
* [Extra docs]()
* Clock: 1000 Hz
* External hardware: Any microcontroller with an SPI peripheral



### How it works

The original QTCore-A1 from Hammond Pearce is a basic accumulator-based 8-bit microarchitecture (with an emphasis on the micro). It is a Von Neumann design (shared data and instruction memory).
Its original specs can be found here: https://github.com/kiwih/tt03-verilog-qtcoreA1

In order to lock the design, I had to make some space so I cut off 2 (8bit) memory registers (as a 16 bit register was added to store the locking key) and the memory mapped constants.
I locked the ALU with 6 bits, locking 1 operation an 3 constants, and the control unit with 10 bits, locking an 8 bit constnt used twice and 2 branches. This locking techniques are the same one first presented in ASSURE: https://arxiv.org/abs/2010.05344

I also locked the ISA to ALU opcodes module using a novel RTL locking technique to lock case statements. 
The case variable is xored with the locking key and all case constants are replaced with the results of the original value xored with the right key.
In this way the case statement works as originally intended only with the right key. 
To implement this locking technique I used ChatGPT with the GPT4 version. It was able to modify the isa_to_alu_opcode module
by describing the technique, the key value to use, and the original module. The structure was correct from the first response, some additional
back and forth was required for it to get the right values for the case constants (apparently gpt4 struggles a bit with xor operations).

The correct key is provided to allow everyone to use this design: 1011 1111 1111 1001
We store the key in a scan chained register. In order to have the design working, your assembly should end with this two lines:

```
15: DATA 249    ; logic locking unlock key 
16: DATA 191    ; logic locking unlock key 
```
### 7seg

Tiny Tapeout 3 has a 7-segment display on the board. To make it useful with the QTCore-A1, 
the processor helpfully includes the bit patterns stored at addresses 16-26. Address 15 contains the Address of the 0 pattern.
The easiest way to make use of these is with the LDAR instruction.
Here's an example snippet, which assumes a value between 0 and 9 is stored at address 13:

```
0: LDA 13 ; load the value we want to display
1: ADD 15 ; add it to the constant 15 to get 
;           the 7 segment pattern offset
2: LDAR   ; load the 7 segment pattern
3: STA 14 ; emit the 7 segment pattern
```


### How to test

The processor will not run unless a program is scanned into it.
Fortunately, this is easy using the SPI peripheral of an external microcontroller.

### Wiring the SPI

The QTCore-A1 is designed to be connected to an SPI peripheral along with two chip selects.
This is because we use the SPI SCK as the clock for both the scan chain and the processor.

Connect the I/O according to the provided wiring table. Then, set the SPI peripheral to the following settings:

* SPI Mode 0
* 8-bit data
* MSB first
* A very slow clock (I used 1kHz)
* The scan chain chip select is active low
* The processor chip select is active low

### Processor Memory Map

| Address | Description |
|---------|-------------|
| 0-13    | 14 bytes of general purpose Instruction/Data memory |
| 14      | I/O: {OUT[7:1], IN[0]} |
| 15      | Constant value: 16 (See "7seg" note below) |
| 16      | Constant value: Bits for 7seg "0" |
| 17      | Constant value: Bits for 7seg "1" |
| 18      | Constant value: Bits for 7seg "2" |
| 19      | Constant value: Bits for 7seg "3" |
| 21      | Constant value: Bits for 7seg "4" |
| 22      | Constant value: Bits for 7seg "5" |
| 23      | Constant value: Bits for 7seg "6" |
| 24      | Constant value: Bits for 7seg "7" |
| 25      | Constant value: Bits for 7seg "8" |
| 26      | Constant value: Bits for 7seg "9" |
| 27-31   | Constant value: 1 |

### Loading a program

During the scan chain mode (when the scan enable is low), the entire processor acts as a giant shift register, with the bits in the following order:

All elements of the scan chain are presented MSB first.

* `scan_chain[2:0]` - 3-bit state register
* `scan_chain[7:3]` - 5-bit PC
* `scan_chain[15:8]` - 8-bit Instruction Register 
* `scan_chain[23:16]` - 8-bit Accumulator 
* `scan_chain[31 -: 8]` - Memory[0]
* `scan_chain[39 -: 8]` - Memory[1]
* `scan_chain[47 -: 8]` - Memory[2]
* `scan_chain[55 -: 8]` - Memory[3]
* `scan_chain[63 -: 8]` - Memory[4]
* `scan_chain[71 -: 8]` - Memory[5]
* `scan_chain[79 -: 8]` - Memory[6]
* `scan_chain[87 -: 8]` - Memory[7]
* `scan_chain[95 -: 8]` - Memory[8]
* `scan_chain[103 -: 8]` - Memory[9]
* `scan_chain[111 -: 8]` - Memory[10]
* `scan_chain[119 -: 8]` - Memory[11]
* `scan_chain[127 -: 8]` - Memory[12]
* `scan_chain[135 -: 8]` - Memory[13]
* `scan_chain[143 -: 8]` - Memory[14](the IO register)
* `scan_chain[159 -: 16]` - (Locking Key)


Example program:

```
; This program tests the btn and LEDs
; will switch between 0 and 7 pressing the button
;
; BTNs and LEDS are at address 17, btn at LSB
0: ADDI 14 ; load the btn and LEDS
1: STA 14 ; load the btn and LEDS
2: LDA 14 ; load the btn and LEDS
3: AND 12 ; mask the btn
4: BNE_BWD ; if btn&1 is not zero then branch back to 0
5: LDA 13 ; load the btn and LEDS
6: STA 14 ; load the btn and LEDS
7: LDA 14 ; load the btn and LEDS
8: AND 12 ; mask the btn
9: BEQ_BWD ; if btn&1 is zero then branch back to 3
;
; the button has now done a transition from low to high
;
10: CLR
11: JMP
;
; data
12: DATA 1;
13: DATA 126 ;
15: DATA 249    ; logic locking unlock key 
16: DATA 191    ; logic locking unlock key 
```


C code to load the previously-discussed example program (e.g. via the STM32 HAL) is provided:
```
//The registers are presented in reverse order to 
// the table as we load them MSB first.
uint8_t program_led_btn[21] = {
          0b10111111, //MEM[16]
          0b11111001, //MEM[15]
          0b00000000, // IOREG
          0b01111110, //MEM[13]
          0b00000001, //MEM[12]
          0b11110000, //MEM[11]
          0b11111101, //MEM[10]
          0b11110011, //MEM[9]
          0b10001100, //MEM[8]
          0b00001110, //MEM[7]
          0b00101110, //MEM[6]
          0b00001101, //MEM[5]
          0b11110101, //MEM[4]
          0b10001100, //MEM[3]
          0b00001110, //MEM[2]
          0b00101110, //MEM[1]
          0b11101110, //MEM[0]
          0b00000000, //ACC
          0b00000000, //IR
          0b00000001 //PC[5bit], CU[3bit]
};

//... this will go in the SPI peripheral initialization code
hspi1.Init.Mode = SPI_MODE_MASTER;
hspi1.Init.Direction = SPI_DIRECTION_2LINES;
hspi1.Init.DataSize = SPI_DATASIZE_8BIT;
hspi1.Init.CLKPolarity = SPI_POLARITY_LOW;
hspi1.Init.CLKPhase = SPI_PHASE_1EDGE;
hspi1.Init.NSS = SPI_NSS_SOFT;
//...
hspi1.Init.FirstBit = SPI_FIRSTBIT_MSB;
hspi1.Init.TIMode = SPI_TIMODE_DISABLE;
//...

//... this will go in your main or similar
uint8_t *program = program_led_btn;
//on the first run, scan_out will give us the reset value of the 
// processor
HAL_GPIO_WritePin(SPI_SCAN_CS_GPIO_Port, SPI_SCAN_CS_Pin, 0);
HAL_Delay(100);
HAL_SPI_TransmitReceive(&hspi1, program, scan_out, 21, HAL_MAX_DELAY);
HAL_Delay(100);
HAL_GPIO_WritePin(SPI_SCAN_CS_GPIO_Port, SPI_SCAN_CS_Pin, 1);
//we can check if the program loaded correctly by immediately scanning 
// it back in again, it will be unloaded to scan_out
HAL_GPIO_WritePin(SPI_SCAN_CS_GPIO_Port, SPI_SCAN_CS_Pin, 0);
HAL_Delay(100);
HAL_SPI_TransmitReceive(&hspi1, program, scan_out, 21, HAL_MAX_DELAY);
HAL_Delay(100);
HAL_GPIO_WritePin(SPI_SCAN_CS_GPIO_Port, SPI_SCAN_CS_Pin, 1);
//Check if the program loaded correctly
for(int i = 0; i < 21; i++) {
  if(program[i] != scan_out[i]) {
    while(1); //it failed
  }
}
```

### Running a program

Once the program is loaded (using the above code or similar), we can run a program.
This is as easy as providing a clock signal to the processor and setting the processor enable line high.
We do this using the SPI as follows:
```
uint8_t dummy; //a dummy value
HAL_GPIO_WritePin(SPI_PROC_CS_GPIO_Port, SPI_PROC_CS_Pin, 0);
//...
//run this in a loop
HAL_SPI_TransmitReceive(&hspi1, &dummy, &dummy, 1, HAL_MAX_DELAY);
```

The processor will ignore any value being shifted in on the MOSI data line during operation.
However, it does provide a nice feature in that the processor will emit the current value of the processor_halt signal on the MISO line.
This means that we can improve the code to run this process in a loop and catch when the program reaches a HLT instruction:
```
HAL_Delay(100);
HAL_GPIO_WritePin(SPI_PROC_CS_GPIO_Port, SPI_PROC_CS_Pin, 0);
dummy = 0;
while(1) {
  HAL_SPI_TransmitReceive(&hspi1, &dummy, &dummy, 1, HAL_MAX_DELAY);
  if(dummy == 0xFF)
    break;
}
HAL_GPIO_WritePin(SPI_PROC_CS_GPIO_Port, SPI_PROC_CS_Pin, 1);
HAL_Delay(100);
```
Of course, the example program does not HLT, so we will not reach this point in this code. But, it works for other programs that do contain a HLT instruction.

Once you have the provided example program running, you will be able to press the button and see how the LEDs are toggled.


### IO

| # | Input        | Output       |
|---|--------------|--------------|
| 0 | clock - connect to an SPI SCK.  | general purpose output 0 (e.g. LED segment a). This output comes from the I/O register bit 1. |
| 1 | reset (active high)  | general purpose output 1 (e.g. LED segment b). This output comes from the I/O register bit 2. |
| 2 | scan enable (active low) - connect to an SPI chip select.  | general purpose output 2 (e.g. LED segment c). This output comes from the I/O register bit 3. |
| 3 | processor enable (active low) - connect to an SPI chip select.  | general purpose output 3 (e.g. LED segment d). This output comes from the I/O register bit 4. |
| 4 | scan data in - connect to an SPI MOSI.  | general purpose output 4 (e.g. LED segment e). This output comes from the I/O register bit 5. |
| 5 | general purpose input (e.g. Button). This input will be provided to the I/O register bit 0.  | general purpose output 5 (e.g. LED segment f). This output comes from the I/O register bit 6. |
| 6 | none  | general purpose output 6 (e.g. LED segment g). This output comes from the I/O register bit 7. |
| 7 | none  | scan data out - connect to an SPI MISO. |
