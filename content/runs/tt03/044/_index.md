---
hidden: true
title: "44 QTCore-A1"
weight: 45
---

## 44 : QTCore-A1

* Author: Hammond Pearce
* Description: An accumulator-based 8-bit microarchitecture designed via GPT-4 conversations.
* [GitHub repository](https://github.com/kiwih/tt03-verilog-qtcoreA1)
* [Most recent GDS build](https://github.com/kiwih/tt03-verilog-qtcoreA1/actions/runs/4763642389)
* HDL project
* [Extra docs]()
* Clock: 1000 Hz
* External hardware: Any microcontroller with an SPI peripheral



### How it works

The QTCore-A1 is a basic accumulator-based 8-bit microarchitecture (with an emphasis on the micro). It is a Von Neumann design (shared data and instruction memory).

Although primarily designed for Tiny Tapeout 3, it is parameterized and may be synthesized to FPGAs (and a project file for CMOD A7 is provided on the project GitHub).

Probably the most interesting thing about this design is that all functional Verilog beyond the Tiny Tapeout wrapper was written by GPT-4, i.e. not a human! 
The author (Hammond Pearce) developed with GPT-4 first the ISA, then the processor, fully conversationally. Hammond wrote the test-benches to validate the design, and then had the
appropriate back-and-forth with GPT-4 to have it fix all bugs.
For your interest, we will provide all conversation logs in the project repository.

The architecture defines a processor with the following components:

* Control Unit: 2-cycle FSM for driving the processor (3 bit one-hot encoded state register)
* Program Counter: 5-bit register containing the current address of the program
* Instruction Register: 8-bit register containing the current instruction to execute
* Accumulator: 8-bit register used for data storage, manipulation, and logic
* Memory Bank: 17 8-bit registers which store instructions and data. The 17th register is used for I/O.

In order to interact with the processor, all registers are connected via one large scan chain.
As such, you can use an external microcontroller's SPI peripheral to read and write the status of the processor.
We use the SPI clock SPI_SCK as the clock to drive both the QTCore-A1 and the scan chain.
You choose which you are using by asserting the appropriate chip select.
This makes it quite easy to interact with the processor.

The ISA description follows. 
For your convenience, we also provide an assembler in Python (also written by GPT-4) which makes it quite easy to write simple programs.

### Immediate Data Manipulation Instructions
- ADDI: Add 4-bit Immediate to Accumulator
  - Opcode (4 bits): 1110
  - Immediate (4 bits): 4-bit Immediate
  - Register Effects: ACC <- ACC + IMM

### Instructions with Variable-Data Operands
- LDA: Load Accumulator with memory contents
  - Opcode (3 bits): 000
  - Operand (5 bits): Memory Address
  - Register Effects: ACC <- M[Address]

- STA: Store Accumulator to memory
  - Opcode (3 bits): 001
  - Operand (5 bits): Memory Address
  - Register Effects: M[Address] <- ACC

- ADD: Add memory contents to Accumulator
  - Opcode (3 bits): 010
  - Operand (5 bits): Memory Address
  - Register Effects: ACC <- ACC + M[Address]

- SUB: Subtract memory contents from Accumulator
  - Opcode (3 bits): 011
  - Operand (5 bits): Memory Address
  - Register Effects: ACC <- ACC - M[Address]

- AND: AND memory contents with Accumulator
  - Opcode (3 bits): 100
  - Operand (5 bits): Memory Address
  - Register Effects: ACC <- ACC & M[Address]

- OR: OR memory contents with Accumulator
  - Opcode (3 bits): 101
  - Operand (5 bits): Memory Address
  - Register Effects: ACC <- ACC | M[Address]

- XOR: XOR memory contents with Accumulator
  - Opcode (3 bits): 110
  - Operand (5 bits): Memory Address
  - Register Effects: ACC <- ACC ^ M[Address]

### Control and Branching Instructions
- JMP: Jump to memory address
  - Opcode (8 bits): 11110000
  - PC Behavior: PC <- ACC (Load the PC with the address stored in the accumulator)

- JSR: Jump to Subroutine (save address to ACC)
  - Opcode (8 bits): 11110001
  - PC Behavior: ACC <- PC + 1, PC <- ACC (Save the next address in ACC, then jump to the address in ACC)

- BEQ_FWD: Branch if equal, forward (branch if ACC == 0)
  - Opcode (8 bits): 11110010
  - PC Behavior: If ACC == 0, then PC <- PC + 3 (Jump 2 instructions forward if ACC is zero)

- BEQ_BWD: Branch if equal, backward (branch if ACC == 0)
  - Opcode (8 bits): 11110011
  - PC Behavior: If ACC == 0, then PC <- PC - 2 (Jump 1 instruction backward if ACC is zero)

- BNE_FWD: Branch if not equal, forward (branch if ACC != 0)
  - Opcode (8 bits): 11110100
  - PC Behavior: If ACC != 0, then PC <- PC + 3 (Jump 2 instructions forward if ACC is non-zero)

- BNE_BWD: Branch if not equal, backward (branch if ACC != 0)
  - Opcode (8 bits): 11110101
  - PC Behavior: If ACC != 0, then PC <- PC - 2 (Jump 1 instruction backward if ACC is non-zero)

- HLT: Halt the processor until reset
  - Opcode (8 bits): 11111111
  - PC Behavior: Stop execution (PC does not change until a reset occurs)

### Data Manipulation Instructions
- SHL: Shift Accumulator left
  - Opcode (8 bits): 11110110
  - Register Effects: ACC <- ACC << 1

- SHR: Shift Accumulator right
  - Opcode (8 bits): 11110111
  - Register Effects: ACC <- ACC >> 1

- SHL4: Shift Accumulator left by 4 bits
  - Opcode (8 bits): 11111000
  - Register Effects: ACC <- ACC << 4

- ROL: Rotate Accumulator left
  - Opcode (8 bits): 11111001
  - Register Effects: ACC <- (ACC << 1) OR (ACC >> 7)

- ROR: Rotate Accumulator right
  - Opcode (8 bits): 11111010
  - Register Effects: ACC <- (ACC >> 1) OR (ACC << 7)

- LDAR: Load Accumulator via indirect memory access (ACC as ptr)
  - Opcode (8 bits): 11111011
  - Register Effects: ACC <- M[ACC]

- DEC: Decrement Accumulator
  - Opcode (8 bits): 11111100
  - Register Effects: ACC <- ACC - 1

- CLR: Clear (Zero) Accumulator
  - Opcode (8 bits): 11111101
  - Register Effects: ACC <- 0

- INV: Invert (NOT) Accumulator
  - Opcode (8 bits): 11111110
  - Register Effects: ACC <- ~ACC

### Example programming using the assembler

Writing assembly programs for QTCore-A1 is simplified by the assembler produced by GPT-4. First, we define two additional meta-instructions:

### Meta-instructions:

- NOP: Do nothing
  - Implemented as ADDI 0

- DATA: Define raw data to be loaded at the current address
  - Operand (8 bits): 8-bit data value

### Presenting programs to the assembler:

1. Programs are presented in the format `[address]: [mnemonic] [optional operand]`
2. There is a special meta-instruction called DATA, which is followed by a number. If this is used, just place that number at that address.
3. Programs cannot exceed the size of the memory (in Tiny Tapeout 3, this is 17 bytes including the IO register).
4. The memory contains both instructions and data.

### Example program:
An interesting example program is presented. This assumes a button is connected to the general purpose input, and some LEDs are connected to the LED output.

We assume this file is called `test_btn_led.asm`:
```
; This program tests the btn and LEDs
; It will wait for low->high transitions on the button input. 
; After receiving this, it will toggle a set of LEDs.
;
; BTNs and LEDS are at address 17, btn at LSB
0: LDA 17 ; load the btn and LEDS
1: AND 16 ; mask the btn
2: BNE_BWD ; if btn&1 is not zero then branch back to 0
3: LDA 17 ; load the btn and LEDS
4: AND 16 ; mask the btn
5: BEQ_BWD ; if btn&1 is zero then branch back to 3
;
; the button has now done a transition from low to high
;
6: LDA 14 ; load the counter toggle
7: XOR 16 ; toggle the counter using the btn mask
8: STA 14 ; store the counter value
9: ADDI 14 ; get the counter value offset 
;            (if 0, will be 14 (which is 0), if 1, 15)
10: LDAR ; load the counter LED pattern
11: STA 17 ; store the LED pattern
12: CLR
13: JMP
;
; data
;
14: DATA 0; toggle and LED pattern 0
15: DATA 24; LED pattern of ON 
;            (test for led_out=value 12, since 24>>1 == 12)
16: DATA 1 ; btn and counter mask
```

To compile this program, we would invoke the assembler (provided in the associated repository) as follows:

```
$ ./assembler.py test_btn_led.asm
```

The assembler will generate the following files for us:

* `test_btn_led.bin`: This is a binary representation of the assembly program. It can be used, or we can use one of the helper formats...
* `test_btn_led.memarray.v`: This provides the binary ready for use in a Verilog test bench to directly write to the memory of the processor.
* `test_btn_led.scanchain.v`: This provides the binary ready for use in the provided Verilog test bench which loads and unloads programs via the scan chain.
* `test_btn_led.c`: This provides the binary as an array suitable for use in a C program on an external microcontroller which can load it into the processor via SPI.

### Processor operation

The processor executes all instructions via 2 stages (multi-cycle).

The timing is as follows. Note the branch instructions are +2/-3 due to the already-incremented PC in the fetch stage.

**FETCH cycle (all instructions)**

1. IR <- M[PC]
2. PC <- PC + 1

**EXECUTE cycle**

For **Immediate Data Manipulation Instructions**:

* ADDI: ACC <- ACC + IMM

For **Instructions with Variable-Data Operands**:

* LDA: ACC <- M[Address]
* STA: M[Address] <- ACC
* ADD: ACC <- ACC + M[Address]
* SUB: ACC <- ACC - M[Address]
* AND: ACC <- ACC & M[Address]
* OR:  ACC <- ACC | M[Address]
* XOR: ACC <- ACC ^ M[Address]

For **Control and Branching Instructions**:

* JMP: PC <- ACC
* JSR: ACC <- PC, PC <- ACC
* BEQ_FWD: If ACC == 0, then PC <- PC + 2
* BEQ_BWD: If ACC == 0, then PC <- PC - 3
* BNE_FWD: If ACC != 0, then PC <- PC + 2
* BNE_BWD: If ACC != 0, then PC <- PC - 3
* HLT: (No operation, processor halted)

For **Data Manipulation Instructions**:

* SHL: ACC <- ACC << 1
* SHR: ACC <- ACC >> 1
* SHL4: ACC <- ACC << 4
* ROL: ACC <- (ACC << 1) OR (ACC >> 7)
* ROR: ACC <- (ACC >> 1) OR (ACC << 7)
* LDAR: ACC <- M[ACC]
* DEC: ACC <- ACC - 1
* CLR: ACC <- 0
* INV: ACC <- ~ACC

### Processor Memory Map

| Address | Description |
|---------|-------------|
| 0-16    | 17 bytes of general purpose Instruction/Data memory |
| 17      | I/O: {OUT[7:1], IN[0]} |
| 18      | Constant value: 19 (See "7seg" note below) |
| 19      | Constant value: Bits for 7seg "0" |
| 20      | Constant value: Bits for 7seg "1" |
| 21      | Constant value: Bits for 7seg "2" |
| 22      | Constant value: Bits for 7seg "3" |
| 23      | Constant value: Bits for 7seg "4" |
| 24      | Constant value: Bits for 7seg "5" |
| 25      | Constant value: Bits for 7seg "6" |
| 26      | Constant value: Bits for 7seg "7" |
| 27      | Constant value: Bits for 7seg "8" |
| 28      | Constant value: Bits for 7seg "9" |
| 29-31   | Constant value: 1 |

### 7seg

Tiny Tapeout 3 has a 7-segment display on the board. To make it useful with the QTCore-A1, 
the processor helpfully includes the bit patterns stored at addresses 19-28.
The easiest way to make use of these is with the LDAR instruction.
Here's an example snippet, which assumes a value between 0 and 9 is stored at address 16:

```
0: LDA 16 ; load the value we want to display
1: ADD 18 ; add it to the constant 19 to get the 7 segment pattern offset
2: LDAR   ; load the 7 segment pattern
3: STA 17 ; emit the 7 segment pattern
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
* `scan_chain[143 -: 8]` - Memory[14]
* `scan_chain[151 -: 8]` - Memory[15]
* `scan_chain[159 -: 8]` - Memory[16]
* `scan_chain[167 -: 8]` - Memory[17] (the IO register)

C code to load the previously-discussed example program (e.g. via the STM32 HAL) is provided:
```
//The registers are presented in reverse order to 
// the table as we load them MSB first.
uint8_t program_led_btn[21] = {
    0b00000000, //IOREG
    0b00000001, //MEM[16]
    0b00011000, //MEM[15]
    0b00000000, //MEM[14]
    0b11110000, //MEM[13]
    0b11111101, //MEM[12]
    0b00110001, //MEM[11]
    0b11111011, //MEM[10]
    0b11101110, //MEM[9]
    0b00101110, //MEM[8]
    0b11010000, //MEM[7]
    0b00001110, //MEM[6]
    0b11110011, //MEM[5]
    0b10010000, //MEM[4]
    0b00010001, //MEM[3]
    0b11110101, //MEM[2]
    0b10010000, //MEM[1]
    0b00010001, //MEM[0]
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
