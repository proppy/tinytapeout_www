---
hidden: true
title: "63 QTChallenges"
weight: 64
---

## 63 : QTChallenges

* Author: Jason Blocklove
* Description: This project implements 8 different benchmark circuits created 100% with ChatGPT-4.
* [GitHub repository](https://github.com/JBlocklove/tt03-qtchallenges-chatgpt_4)
* [Most recent GDS build](https://github.com/JBlocklove/tt03-qtchallenges-chatgpt_4/actions/runs/4803094867)
* HDL project
* [Extra docs]()
* Clock: 15000 Hz
* External hardware: 



### How it works

This design implements a series of 8 benchmark circuits selectable by 3 bits of the design input, all of which written by ChatGPT-4. A series of prompts for each circuit were created which had ChatGPT design the module itself as well as a Verilog testbench for the design, and the design was considered finalized when there were no errors from simulation or synthesis. As much of the feedback as possible was given by the tools -- [Icarus Verilog](https://github.com/steveicarus/iverilog) for simulation and the Tiny Tapeout OpenLane/yosys toolchain for synthesis.

As a result of the designs being 100% created by ChatGPT, they all passed their ChatGPT-created testbenches, but several are not functionally correct as the generated testbenches are insufficient or incorrect. The best example of this is the **Dice Roller** benchmark, which has a constant output but passes its testbench fully.

The only design with a human-made testbench is the **Wrapper Module**, whose testbench uses dummy modules just to ensure the multiplexing works as expected. It seemed unrealistic to have ChatGPT create a testbench for every benchmark all in one given the token limits and how it struggled to make some of the standalone testbenches.

The complete transcripts of the ChatGPT conversations can be found at https://github.com/JBlocklove/tt03-chatgpt-4_benchmarks/tree/main/conversations

---

#### Wrapper Module/Multiplexer

### ChatGPT Prompt
```
I am trying to create a Verilog model for a wrapper around several
benchmarks, sepecifically for the Tiny Tapeout project. It must meet
the following specifications:
    - Inputs:
        - io_in (8-bits)
    - Outputs:
        - io_out (8-bits)

The design should instantiate the following modules, and use 3
bits of the 8-bit input to select which one will output from the
module.

Benchmarks:
    - shift_register:
	    - Inputs:
	    	- clk
	    	- reset_n
	    	- data_in
	    	- shift_enable
	    - Outputs:
	    	- [7:0] data_out

    - sequence_generator:
	    - Inputs:
	    	- clock
	    	- reset_n
	    	- enable
	    - Outputs:
	    	- [7:0] data

    - sequence_detector:
	    - Inputs:
	    	- clk
	    	- reset_n
	    	- [2:0] data
	    - Outputs:
	    	- sequence_found

    - abro_state_machine:
        - Inputs:
            - clk
            - reset_n
            - A
            - B
        - Outputs:
            - O
            - [3:0] state

    - binary_to_bcd:
	    - Inputs:
	    	- [4:0] binary_input
	    - Outputs:
	    	- [3:0] bcd_tens
            - [3:0] bcd_units

    - lfsr:
	    - Inputs:
	    	- clk
            - reset_n
	    - Outputs:
	    	- [7:0] data

    - traffic_light:
        - Inputs:
            - clk
            - reset_n
            - enable
        - Outputs:
            - red
            - yellow
            - green

    - dice_roller:
        - Inputs:
            - clk
            - reset_n
            - [1:0] die_select
            - roll
        - Outputs:
            - rolled_number

How would I write a design that meets these specifications?
```

### Benchmark I/O Mapping

| `io_in[7:5]`   | Benchmark          |
|---------------:|:-------------------|
| `000`          | Shift Register     |
| `001`          | Sequence Generator |
| `010`          | Sequence Detector  |
| `011`          | ABRO               |
| `100`          | Binary to BCD      |
| `101`          | LFSR               |
| `110`          | Traffic Light      |
| `111`          | Dice Roller        |

### Expected Functionality

The top level module for the design is a wrapper module/multiplexer which allows the user to select which benchmark is being used for the output of the design. Using `io_in[7:5]`, the user can select which benchmark will output to the `io_out` pins.

This module was created after all of the other designs were finalized so their port mappings could be given

### Actual Functionality

The module functions as intended. This is the only module with a human-written testbench, as it seemed unrealistic to have ChatGPT create a full testbench that confirmed the module instantiations worked given how much it struggled with some of the other testbenches.

---

#### Shift Register

### ChatGPT Prompt

```
I am trying to create a Verilog model for a shift register. It
must meet the following specifications:
	- Inputs:
		- Clock
		- Active-low reset
		- Data (1 bit)
		- Shift enable
	- Outputs:
		- Data (8 bits)

How would I write a design that meets these specifications?
```

### Benchmark I/O Mapping

| # | Input           | Output             |
|---|-----------------|--------------------|
| 0 | `clk`           | Shifted data [0]   |
| 1 | `rst_n` (async) | Shifted data [1]   |
| 2 | `data_in`       | Shifted data [2]   |
| 3 | `shift_enable`  | Shifted data [3]   |
| 4 | Not used        | Shifted data [4]   |
| 5 | Select bit      | Shifted data [5]   |
| 6 | Select bit      | Shifted data [6]   |
| 7 | Select bit      | Shifted data [7]   |

### Expected Functionality

The expected functionality of this shift register module is to shift the `data_in` bit in on the right side of the data vector on any rising `clk` edge where `shift_enable` is high.

### Actual Functionality

The module seems to function as intended.

---

#### Sequence Generator

### ChatGPT Prompt
```
I am trying to create a Verilog model for a sequence generator. It
must meet the following specifications:
	- Inputs:
		- Clock
		- Active-low reset
		- Enable
	- Outputs:
		- Data (8 bits)

While enabled, it should generate an output sequence of the
following hexadecimal values and then repeat:
	- 0xAF
	- 0xBC
	- 0xE2
	- 0x78
	- 0xFF
	- 0xE2
	- 0x0B
	- 0x8D

How would I write a design that meets these specifications?
```

### Benchmark I/O Mapping

| # | Input           | Output                |
|---|-----------------|-----------------------|
| 0 | `clk`           | Sequence Output [0]   |
| 1 | `rst_n` (async) | Sequence Output [1]   |
| 2 | Not used        | Sequence Output [2]   |
| 3 | Not used        | Sequence Output [3]   |
| 4 | `enable`        | Sequence Output [4]   |
| 5 | Select bit      | Sequence Output [5]   |
| 6 | Select bit      | Sequence Output [6]   |
| 7 | Select bit      | Sequence Output [7]   |

### Expected Functionality

The expected functionality of this sequence generator is to output the following sequence, moving a step forward whenever the `clk` has a rising edge and the `enable` is high. Once the sequence has reached its end it should repeat.

```
0xAF
0xBC
0xE2
0x78
0xFF
0xE2
0x0B
0x8D
```

### Actual Functionality

The module functions as intended.

---

#### Sequence Detector

### ChatGPT Prompt
```
I am trying to create a Verilog model for a sequence detector. It
must meet the following specifications:
	- Inputs:
		- Clock
		- Active-low reset
		- Data (3 bits)
	- Outputs:
		- Sequence found

While enabled, it should detect the following sequence of binary
input values:
	- 0b001
	- 0b101
	- 0b110
	- 0b000
	- 0b110
	- 0b110
	- 0b011
	- 0b101

How would I write a design that meets these specifications?
```

### Benchmark I/O Mapping

| # | Input           | Output                |
|---|-----------------|-----------------------|
| 0 | `clk`           | Not Used              |
| 1 | `rst_n` (async) | Not Used              |
| 2 | `data[0]`       | Not Used              |
| 3 | `data[1]`       | Not Used              |
| 4 | `data[2]`       | Not Used              |
| 5 | Select bit      | Not Used              |
| 6 | Select bit      | Not Used              |
| 7 | Select bit      | Sequence Found        |

### Expected Functionality

The expected functionality of this sequence detector is to output a `1` if it receives the following sequence of data all on consecutive clock cycles.

```
0b001
0b101
0b110
0b000
0b110
0b110
0b011
0b101
```

### Actual Functionality

The module does not correctly detect the sequence. In trying to set the states to allow the sequence to overlap it instead skips the final value or outputs a `1` if the second to last value and final value are both `0b101`.

---

#### ABRO State Machine

### ChatGPT Prompt
```
I am trying to create a Verilog model for an ABRO state machine.
It must meet the following specifications:
    - Inputs:
        - Clock
        - Active-low reset
        - A
        - B
    - Outputs:
        - O
        - State

Other than the main output from ABRO machine, it should output the
current state of the machine for use in verification.

The states for this state machine should be one-hot encoded.

How would I write a design that meets these specifications?
```

### Benchmark I/O Mapping

| # | Input             | Output                |
|---|-------------------|-----------------------|
| 0 | `clk`             | State [0]             |
| 1 | `reset_n` (async) | State [1]             |
| 2 | `A`               | State [2]             |
| 3 | `B`               | State [3]             |
| 4 | Not used          | Output                |
| 5 | Select bit        | Not used              |
| 6 | Select bit        | Not used              |
| 7 | Select bit        | Not used              |

### Expected Functionality

The expected functionality of the ABRO (A, B, Reset, Output) state machine is to only reach the output state and output a `1` when both inputs `A` and `B` have been given before a reset. The order of the inputs should not matter, so long as both `A` and `B` are set.

### Actual Functionality

The module does not function fully as intended. If `B` is received before `A` then it works as intended, but if `A` is received first then it actually requires the sequence `A, B, A` in order to reach the output state. It also does not handle the case where `A` and `B` are set in the same cycle, instead interpreting it as if `A` was received first.

---

#### Binary to BCD Converter

### ChatGPT Prompt
```
I am trying to create a Verilog model for a binary to
binary-coded-decimal converter. It must meet the following
specifications:
	- Inputs:
		- Binary input (5-bits)
	- Outputs:
		- BCD (8-bits: 4-bits for the 10's place and 4-bits for the 1's place)

How would I write a design that meets these specifications?
```

### Benchmark I/O Mapping

| # | Input             | Output         |
|---|-------------------|----------------|
| 0 | `binary_input[0]` | BCD Ones [0]   |
| 1 | `binary_input[1]` | BCD Ones [1]   |
| 2 | `binary_input[2]` | BCD Ones [2]   |
| 3 | `binary_input[3]` | BCD Ones [3]   |
| 4 | `binary_input[4]` | BCD Tens [0]   |
| 5 | Select bit        | BCD Tens [1]   |
| 6 | Select bit        | BCD Tens [2]   |
| 7 | Select bit        | BCD Tens [3]   |

### Expected Functionality

The expected functionality of this module is to take a 5-bit binary number and produce a binary-coded-decimal output. The 4 most significant bits of the output encode to the tens place of the decimal number, the 4 least signification bits of the output encode the ones place of the decimal number

### Actual Functionality

The module functions as intended.

---

#### Linear Feedback Shift Register (LFSR)

### ChatGPT Prompt
```
I am trying to create a Verilog model for an LFSR. It must meet
the following specifications:
	- Inputs:
		- Clock
        - Active-low reset
	- Outputs:
		- Data (8-bits)

The initial state should be 10001010, and the taps should be at
locations 1, 4, 6, and 7.

How would I write a design that meets these specifications?
```

### Benchmark I/O Mapping

| # | Input           | Output            |
|---|-----------------|-------------------|
| 0 | `clk`           | Data Output [0]   |
| 1 | `rst_n` (async) | Data Output [1]   |
| 2 | Not used        | Data Output [2]   |
| 3 | Not used        | Data Output [3]   |
| 4 | Not used        | Data Output [4]   |
| 5 | Select bit      | Data Output [5]   |
| 6 | Select bit      | Data Output [6]   |
| 7 | Select bit      | Data Output [7]   |

### Expected Functionality

The expected functionality of this module is to generate a pseudo-random output value from this LFSR based on the tap locations given in the prompt.

It was unspecified in the prompt, but originally it was expected that the LFSR would shift right as is standard amongst most documentation. It instead shifts to the left, which is not inherently incorrect but warrants mentioning.

### Actual Functionality

This module functions almost as expected, except the taps were placed on indices off-by-one. Rather than being at indices 1, 4, 6, and 7, they are at indices 0, 3, 5, and 6. This is still a valid LFSR, it is just not quite was was requested.

---

#### Traffic Light State Machine

### ChatGPT Prompt
```
I am trying to create a Verilog model for a traffic light state
machine. It must meet the following specifications:
    - Inputs:
        - Clock
        - Active-low reset
        - Enable
    - Outputs:
        - Red
        - Yellow
        - Green

The state machine should reset to a red light, change from red to
green after 32 clock cycles, change from green to yellow after 20
clock cycles, and then change from yellow to red after 7 clock
cycles.

How would I write a design that meets these specifications?
```

### Benchmark I/O Mapping

| # | Input           | Output                |
|---|-----------------|-----------------------|
| 0 | `clk`           | Sequence Output [0]   |
| 1 | `rst_n` (async) | Sequence Output [1]   |
| 2 | Not used        | Sequence Output [2]   |
| 3 | `enable`        | Sequence Output [3]   |
| 4 | Not used        | Sequence Output [4]   |
| 5 | Select bit      | Green                 |
| 6 | Select bit      | Yellow                |
| 7 | Select bit      | Red                   |

### Expected Functionality

The expected functionality of this module is to simulate the function of a timed traffic light. On a reset it outputs a red light, waits 32 clock cycles and then changed to a green light, waits 20 clock cycles and then changes to a yellow light, waits 7 clock cycles and then changes back to red. This should then repeat. If the enable is low, then it should pause the operation entirely and pick up again once the enable is brought high again.

### Actual Functionality

The module functions as intended.

---

#### Dice Roller

### ChatGPT Prompt
```
I am trying to create a Verilog model for a simulated dice roller.
It must meet the following specifications:
    - Inputs:
        - Clock
        - Active-low reset
        - Die select (2-bits)
        - Roll
    - Outputs:
        - Rolled number (up to 8-bits)

The design should simulate rolling either a 4-sided, 6-sided,
8-sided, or 20-sided die, based on the input die select. It should
roll when the roll input goes high and output the random number
based on the number of sides of the selected die.

How would I write a design that meets these specifications?
```

### Benchmark I/O Mapping

| # | Input             | Output        |
|---|-------------------|---------------|
| 0 | `clk`             | Dice Roll [0] |
| 1 | `rst_n` (async)   | Dice Roll [1] |
| 2 | `die_select[1]`   | Dice Roll [2] |
| 3 | `die_select[0]`   | Dice Roll [3] |
| 4 | `roll`            | Dice Roll [4] |
| 5 | Select bit        | Dice Roll [5] |
| 6 | Select bit        | Dice Roll [6] |
| 7 | Select bit        | Dice Roll [7] |

### Expected Functionality

The expected functionality of this dice roller is to allow the user to select which die they would like to simulate rolling based on the following table:

| `die_select` | Number of sides |
|-------------:|:----------------|
| `00`         | 4               |
| `01`         | 6               |
| `10`         | 8               |
| `11`         | 20              |

When `roll` is high the module should output a new pseudo-random value in the range `[1 - Number of sides]`

### Actual Functionality

This module outputs `2` for the first two dice rolls and then consistently outputs a `1` regardless of what die is selected.


### How to test


Testing this design has some difficulties as there are several different functionalities that are selected between and not all work as expected.

For sequential designs the input vectors are expected to be given across consecutive clock cycles, as such for those designs `io_in[0]` will not be given. For designs with input bits that are unused, those positions will be shown as `x` to represent don't-cares.

The following tables give input test vectors and their expected outputs:

#### Shift Register

| Given Input | Expected Output | Comment    |
|-------------|-----------------|------------|
|`000x000`    |`00000000`       | Reset      |
|`000x001`    |`00000000`       | Disabled   |
|`000x011`    |`00000000`       | Shift in 0 |
|`000x111`    |`00000001`       | Shift in 1 |
|`000x111`    |`00000011`       | Shift in 1 |
|`000x101`    |`00000110`       | Shift in 0 |
|`000x111`    |`00001101`       | Shift in 1 |
|`000x011`    |`00001101`       | Disabled   |
|`000x110`    |`00000000`       | Reset      |

#### Sequence Generator

| Given Input | Expected Output | Comment  |
|-------------|-----------------|----------|
|`0010xx0`    |`10101111`       | Reset    |
|`0010xx1`    |`10101111`       | Disabled |
|`0011xx1`    |`10101111`       | Enabled  |
|`0011xx1`    |`10111100`       | Enabled  |
|`0011xx1`    |`11100010`       | Enabled  |
|`0011xx1`    |`01111000`       | Enabled  |
|`0011xx1`    |`11111111`       | Enabled  |
|`0011xx1`    |`11100010`       | Enabled  |
|`0011xx1`    |`00001011`       | Enabled  |
|`0010xx1`    |`10001101`       | Disabled |
|`0011xx1`    |`10001101`       | Enabled  |
|`0011xx0`    |`10101111`       | Reset    |

#### Sequence Detector

This module does not function as intended. These vectors are meant to represent what will be expected from the hardware, not what is expected from the initial design description.

| Given Input | Expected Output | State     |
|-------------|-----------------|-----------|
|`0100000`    |`00000000`       | S0        |
|`0100001`    |`00000000`       | S0        |
|`0100011`    |`00000000`       | S0 -> S1  |
|`0101011`    |`00000000`       | S1 -> S2  |
|`0101101`    |`00000000`       | S2 -> S3  |
|`0100001`    |`00000000`       | S3 -> S4  |
|`0101101`    |`00000000`       | S4 -> S5  |
|`0101101`    |`00000000`       | S5 -> S6  |
|`0100111`    |`10000000`       | S6 -> S0  |
|`0101011`    |`00000000`       | S0        |
|`0100001`    |`00000000`       | S0        |
|`0100011`    |`00000000`       | S0 -> S1  |
|`0101011`    |`00000000`       | S1 -> S2  |
|`0101101`    |`00000000`       | S2 -> S3  |
|`0100001`    |`00000000`       | S3 -> S4  |
|`0101101`    |`00000000`       | S4 -> S5  |
|`0101101`    |`00000000`       | S5 -> S6  |
|`0101011`    |`10000000`       | S6 -> S7  |
|`0100111`    |`00000000`       | S7 -> S2  |

#### ABRO State Machine

| Given Input | Expected Output | State     |
|-------------|-----------------|-----------|
|`011x000`    |`00000001`       | IDLE      |
|`011x001`    |`00000001`       | IDLE      |
|`011x011`    |`00000010`       | IDLE -> A |
|`011x101`    |`00000100`       | A -> B    |
|`011x111`    |`00011000`       | B -> O    |
|`011x111`    |`00000001`       | O -> IDLE |
|`011x001`    |`00000001`       | IDLE      |
|`011x101`    |`00000100`       | IDLE -> B |
|`011x011`    |`00000100`       | B -> O    |
|`011x011`    |`00000001`       | O -> IDLE |

#### Binary to BCD Converter

This design is purely combinational, so all 8-bits are included in the input vector. Clock cycles do not matter for this functionality.

| Given Input  | Expected Output| Comment |
|--------------|----------------|---------|
|`10000001`    |`00000001`      | 1       |
|`10011111`    |`00110001`      | 31      |
|`10011010`    |`00100110`      | 26      |
|`10010011`    |`00011001`      | 19      |

#### LFSR

As the LFSR only has a clock and reset input, the table will only give the first vectors for resetting and then setting, but it will show the first several cycles of the LFSR from the initial vector.

| Given Input | Expected Output| Comment |
|-------------|----------------|---------|
|`0000000`    |`10001010`      |         |
|`0000001`    |`00010101`      |         |
| ...         |`00101011`      |         |
| ...         |`01010111`      |         |
| ...         |`10101110`      |         |
| ...         |`01011100`      |         |
| ...         |`10111001`      |         |

#### Traffic Light State Machine

Given the nature of this benchmark being dependent on counting clock cycles, it seems unhelpful to provide a suite of test vectors and their expected outputs. A limited set of vectors is provided with comments on their general functionality, but full testing will be left to the user's discretion.

| Given Input | Expected Output| Comment |
|-------------|----------------|---------|
|`110x0x0`    | `10000000`      | Reset: Sets the state machine back to the initial (RED) state |
|`110x0x1`    | Depends      | Disabled: Holds the FSM in its current state, pauses clock cycle count |
|`110x1x1`    | Depends      | Enabled: Resumes clock cycle counting and allows state transitions        |

#### Dice Roller

This module does not function as intended. These vectors are meant to represent what will be expected from the hardware, not what is expected from the initial design description.

| Given Input | Expected Output| Comment |
|-------------|----------------|---------|
|`1110000`    |`00000000`      | Reset   |
|`1111001`    |`00000010`      | d4      |
|`1110001`    |`00000010`      | d4      |
|`1111101`    |`00000001`      | d6      |
|`1110101`    |`00000001`      | d6      |
|`1111011`    |`00000001`      | d8      |
|`1110011`    |`00000001`      | d8      |
|`1111111`    |`00000001`      | d20     |
|`1110111`    |`00000001`      | d20     |
|`1111001`    |`00000001`      | d4      |
|`1111001`    |`00000001`      | d4      |


### IO

| # | Input        | Output       |
|---|--------------|--------------|
| 0 | Depends on benchmark  | Depends on benchmark |
| 1 | Depends on benchmark  | Depends on benchmark |
| 2 | Depends on benchmark  | Depends on benchmark |
| 3 | Depends on benchmark  | Depends on benchmark |
| 4 | Depends on benchmark  | Depends on benchmark |
| 5 | benchmark_select[0]  | Depends on benchmark |
| 6 | benchmark_select[1]  | Depends on benchmark |
| 7 | benchmark_select[2]  | Depends on benchmark |
