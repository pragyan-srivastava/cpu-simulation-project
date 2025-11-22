# 📜 Technical Documentation: The Sum-Till-N CPU

Welcome to the deep-dive documentation for our custom 16-bit CPU! This document contains everything you need to know about the architecture, instruction set, and inner workings of this processor. Let's get started!

---

## 🎯 CPU Overview and Mission

This CPU was designed with a clear purpose: to be a learning tool and a demonstration of digital logic design using the **TkGate simulator**. Its primary mission is to calculate the **Sum of N Natural Numbers**.

It's a 16-bit RISC-like architecture featuring a simple, yet effective, instruction set. While not a commercial-grade processor, it's a powerful educational model that showcases the core concepts of computer architecture.

### Key Specifications

| Feature | Specification |
|---|---|
| 📐 **Architecture** | 16-bit |
| 🛣️ **Data Path** | 16-bit |
| 📜 **Instruction Length** | 16-bit |
| 🔢 **Data Range** | 0 to 65,535 |
| 🖥️ **Registers** | 4 General-Purpose Registers (R0-R3) |
| ⏱️ **Clock Frequency** | 200Hz |
| 🗄️ **Max Instructions** | 128 |
| 🔢 **Program Counter (PC)** | 8-bit |

---

## ⚙️ Staging and Execution Flow

The CPU operates on a 4-stage execution model. While not a true pipeline (as there's no parallel instruction execution), it does use inter-stage buffers to hold results between stages. This is a great pre-cursor to understanding pipelining!

*   **Stage 1: Fetch + Write Back**
    *   The `Program Counter` provides the address.
    *   The `Instruction Fetch` unit grabs the instruction from memory.
    *   The result from the previous operation is written back to the register file.

*   **Stage 2: Decode**
    *   The `Instruction Decode` unit breaks down the instruction into its constituent parts (opcode, operands, etc.).

*   **Stage 3: Value Fetch (Decode 2)**
    *   The values from the registers specified in the instruction are fetched from the `Register File`.

*   **Stage 4: Execute**
    *   The `ALU` or other units perform the actual operation.

### Inter-Stage Buffers (ISBs)
These are non-accessible registers that hold data as it moves between stages:
*   **Stage 1-2:** 1 Register
*   **Stage 2-3:** 7 Registers
*   **Stage 3-4:** 3 Registers
*   **Stage 4-1:** 1 Register

---

## 🧠 Instruction Set Architecture (ISA)

The ISA is the heart of the CPU. It defines the language the processor understands.

### Instruction Format

Our CPU uses a 16-bit fixed-length instruction format, broken down as follows:

| Field | Opcode | Operand 1 | Addr. Mode | Operand 2 / Immediate |
|---|---|---|---|---|
| **Bits** | `[15:12]` (4) | `[11:10]` (2) | `[9:8]` (2) | `[7:0]` (8) |
| **Name** | `WWWW` | `XX` | `YY` | `ZZZZZZZZ` |

*   **WWWW (Opcode):** The main operation to be performed (e.g., ADD, MOV).
*   **XX (Operand 1):** The address of the first register.
*   **YY (Addressing Mode):** How to interpret the second operand.
*   **ZZZZZZZZ (Operand 2/Immediate):** This field is multi-purpose:
    *   If **Register Mode** is used, the first 2 bits (`[7:6]`) are the address of the second register. The other 6 bits are ignored.
    *   If **Immediate Mode** is used, this entire 8-bit field is treated as an immediate value (0-255).
    *   If **User Input Mode** is used, this field is ignored.

### Addressing Modes

| Code (YY) | Mode | Description |
|---|---|---|
| `00` | Register | The second operand is in a register. |
| `01` | Immediate | The second operand is an 8-bit immediate value. |
| `10` | Memory Address | (Not used in this implementation) |
| `11` | User Input | The second operand is taken from user input. |

### Register Set

There are four 16-bit general-purpose registers.

| Register | Encoding |
|---|---|
| R0 | `00` |
| R1 | `01` |
| R2 | `10` |
| R3 | `11` |

### Instruction Table

| Mnemonic | Opcode (WWWW) | Operation | Description |
|---|---|---|---|
| `ADD` | `0000` | `[R1] <- R1 + R2` | Adds contents of R1 and R2, stores in R1. |
| `MUL` | `0001` | `[R1] <- R1 * R2` | Multiplies R1 and R2, stores in R1. |
| `DIV` | `0010` | `[R1] <- R1 / R2` | Divides R1 by R2, stores in R1. |
| `SUB` | `0011` | `[R1] <- R1 - R2` | Subtracts R2 from R1, stores in R1. |
| `MOV` | `0100` | `[R1] <- R2` | Copies the value from R2 to R1. |
| `OUT` | `0101` | `[R1]` | Displays the value of R1. |
| `CMPZ`| `0110` | `[R1] == 0` | Compares R1 to zero and sets a status flag. |
| `JMP` | `0111`| `[PC] <- [PC] + offset` | Jumps to a new address. |
| `STORE`| `1000`| `[R2] <- R1` | Stores R1 in memory (Unused). |
| `LOAD`| `1001`| `[R1] <- [R2]` | Loads from memory into R1 (Unused). |
| `HLT` | `1111` | (Stops CPU) | Halts the processor. |

---

## 💻 Example Code: Sum of N Numbers

Here are two ways the CPU can be programmed to solve the "Sum of N" problem.

### 1. Using Branching (The Looping Method)

This method iteratively adds numbers from N down to 1.

| Hex Addr | Instruction | Binary |
|---|---|---|
| `0x00` | `MOV R0, N` | `0100000100000000` |
| `0x04` | `MOV R1, #0` | `0100010100000000` |
| `0x08` | `MOV R2, #1` | `0100100100000001` |
| `0x0C` | `L1: ADD R1, R0` | `0000010000000000` |
| `0x10` | `SUB R0, R2` | `0011000010000000` |
| `0x14` | `CMPZ R0` | `0110000000000000` |
| `0x18` | `JNE L1` | `0111000100001100` |
| `0x1C` | `OUT R1` | `0101010000000000` |
| `0x20` | `HLT` | `1111111111111111` |

### 2. Without Using Branching (The Formula Method)

This method uses the formula `N * (N + 1) / 2`.

| Hex Addr | Instruction | Binary |
|---|---|---|
| `0x00` | `MOV R0, N` | `0100001100000000` |
| `0x04` | `MOV R2, #2` | `0100100100000010` |
| `0x08` | `MOV R1, R0` | `0100100100000001` |
| `0x0C` | `MUL R0, R0` | `0001000000000000` |
| `0x10` | `ADD R0, R1` | `0000000010000000` |
| `0x14` | `DIV R0, R2` | `0010000010000000` |
| `0x18` | `OUT R0` | `0101000000000000` |
| `0x1C` | `HLT` | `1111111111111111` |

---

## 🧩 Module Breakdown

Here's a closer look at the key modules in the CPU design.

### Program Counter (PC)
*   **Description:** A simple counter that keeps track of the next instruction to be fetched.
*   **Inputs:** `Offset_EN`, `Offset_IN` (from Jump Module), `CLR`, `CLK`.
*   **Outputs:** `PC_out` (the address of the next instruction).

### Instruction Fetch
*   **Description:** Fetches the 16-bit instruction from the Read-Only Memory (ROM).
*   **Inputs:** `PC_IN` (from PC), `EN` (Enable).
*   **Outputs:** `IR_OUT` (the binary instruction).

### Instruction Decoder
*   **Description:** The "brain" of the operation. It takes the raw instruction and generates all the control signals for the other modules.
*   **Inputs:** `IR_IN` (from Instruction Fetch).
*   **Outputs:** `ALU_MODE`, `ALU_EN`, `Reg1`, `Reg2`, `JMP_EN`, `HLT`, and many more!

### Control Unit (CU)
*   **Description:** Manages the overall state of the CPU.
*   **Inputs:** `EN_Decoder`, `CLK`, `CLR`.
*   **Outputs:** Control signals for enabling different stages.
*   **Note:** Contains unused inputs/outputs for a potential future pipelined design (`OP1`, `OP2`, `OP_Forwarding`).

### Register File
*   **Description:** A collection of four 16-bit registers for fast, temporary storage.
*   **Inputs:** `Addr1`, `Addr2`, `RW1` (Read/Write mode for Addr1), `CLR`, `CLK`.
*   **Outputs:** `Data_out1`, `Data_out2`.
*   **Note:** Register at `Addr2` is always in Read mode.

### Arithmetic Logic Unit (ALU)
*   **Description:** The number-cruncher. It performs all arithmetic operations.
*   **Inputs:** `IN1`, `IN2`, `MODE` (ADD, SUB, etc.), `EN`.
*   **Outputs:** `OUT` (the result), `OF` (Overflow Flag).

### 16-bit Comparator
*   **Description:** Compares two 16-bit values. Used for the `CMPZ` instruction.
*   **Inputs:** `REG1`, `REG2`, `EN`.
*   **Outputs:** `OUT` (the comparison result).

### Jump Module
*   **Description:** Calculates the new PC address for a jump instruction.
*   **Inputs:** `JNE_Enable`, `Status_Flag` (from the comparator).
*   **Outputs:** `Offset_Enable`.

---

## 🔬 Program Execution Flow: A Walkthrough

Let's trace the "Sum of N with Branching" program to see how the CPU works its magic. Imagine `N=3`.

1.  **`MOV R0, N`**: `N` (let's say 3) is loaded into register `R0`.
2.  **`MOV R1, #0`**: The immediate value `0` is loaded into `R1`. This will be our accumulator for the sum.
3.  **`MOV R2, #1`**: The immediate value `1` is loaded into `R2`. This is the value we'll subtract in each loop.
4.  **`L1: ADD R1, R0`**: The core of our loop starts here.
    *   The values from `R1` (0) and `R0` (3) are sent to the ALU.
    *   The ALU adds them (`0 + 3 = 3`).
    *   The result (`3`) is written back to `R1`. `R1` is now 3.
5.  **`SUB R0, R2`**:
    *   The values from `R0` (3) and `R2` (1) go to the ALU.
    *   The ALU subtracts them (`3 - 1 = 2`).
    *   The result (`2`) is written back to `R0`. `R0` is now 2.
6.  **`CMPZ R0`**:
    *   The value of `R0` (2) is sent to the Comparator.
    *   The Comparator checks if `R0` is zero. It's not. The status flag is not set.
7.  **`JNE L1`**:
    *   The `Jump Module` checks the status flag. Since the flag is not set (JNE = Jump if Not Equal to zero), it tells the `Program Counter` to jump back to the address of label `L1`.
8.  **`ADD R1, R0`** (Second loop):
    *   `R1` (3) and `R0` (2) are added. `R1` becomes 5.
9.  **`SUB R0, R2`**:
    *   `R0` (2) and `R2` (1) are subtracted. `R0` becomes 1.
10. **`CMPZ R0`**: `R0` (1) is not zero. Status flag is not set.
11. **`JNE L1`**: We jump back to `L1` again.
12. **`ADD R1, R0`** (Third loop):
    *   `R1` (5) and `R0` (1) are added. `R1` becomes 6.
13. **`SUB R0, R2`**:
    *   `R0` (1) and `R2` (1) are subtracted. `R0` becomes 0.
14. **`CMPZ R0`**:
    *   `R0` (0) is now zero! The Comparator sets the status flag.
15. **`JNE L1`**:
    *   The `Jump Module` checks the status flag. It's set, so the condition for the jump is false. The PC increments normally instead of jumping.
16. **`OUT R1`**: The final value of `R1` (6) is displayed.
17. **`HLT`**: The CPU halts. The mission is complete!

---

## 👨‍🏫 Core CPU Theory: The Fetch-Decode-Execute Cycle

Think of the CPU as a master chef in a kitchen. The recipe book is the program in memory (`sumtillN.mem`). The chef's process for making a dish is the **Fetch-Decode-Execute cycle**.

1.  **Fetch (Get the recipe step)**
    *   **Chef:** "What's the next step in the recipe?"
    *   **CPU:** The `Program Counter` (PC) holds the address of the next instruction. The `Instruction Fetch` unit goes to that address in memory and grabs the 16-bit instruction.

2.  **Decode (Understand the step)**
    *   **Chef:** "Ah, it says 'Dice the onions'." The chef knows what "dice" means and what "onions" are.
    *   **CPU:** The `Instruction Decode` unit takes the 16-bit instruction and figures out what it means. "This is an `ADD` operation (`0000`), it uses `R1` and `R2`, and it's a register-to-register operation." The `Control Unit` then sends signals to all the other parts of the CPU, like a sous-chef shouting orders in the kitchen.

3.  **Execute (Do the action)**
    *   **Chef:** *Dices the onions.*
    *   **CPU:** The `ALU` performs the operation. It takes the values from the `Register File`, does the math (or logic), and produces a result. If it's a `JMP` instruction, the `Jump Module` gets involved. If it's a comparison, the `Comparator` does its job.

This cycle repeats over and over, at a blistering 200 times per second (200Hz), until a `HLT` instruction tells the chef to clean up and go home.

---

## 🧐 Architectural Analysis

This CPU design is a fantastic example of a minimal, yet functional, processor. Here's a brief analysis of its structure and design choices.

### Strengths
*   **Clarity and Simplicity:** The design is clean and easy to understand. Because it was built in TkGate, you can visually trace every wire and see how modules connect. This is invaluable for learning.
*   **Purpose-Built:** The CPU is highly optimized for its specific task. There's no wasted complexity.
*   **Good Foundation:** The architecture includes all the standard components of a simple CPU. This makes it an excellent starting point for more advanced features. The unused pipelining logic in the Control Unit is a great example of this foresight.

### Areas for Improvement and Exploration
*   **Pipelining:** The current multi-stage design with inter-stage buffers is a perfect candidate for pipelining. Implementing a true pipeline would involve managing hazards (data, control, and structural) but would provide a massive performance boost.
*   **More Powerful ISA:** The instruction set could be expanded. More addressing modes, more logical operations (AND, OR, XOR), or stack-related instructions (`PUSH`, `POP`) would make it a more general-purpose processor.
*   **Interrupts:** For a more advanced design, adding an interrupt handling mechanism would be a great challenge.

---

## ✨ Final Remarks

This project successfully demonstrates the fundamentals of computer architecture in a clear and interactive way. It's a reminder that the complex processors in our modern devices are all built on these core principles of fetching, decoding, and executing instructions. Whether you're a student of computer science or just a curious tinkerer, this project offers a unique and engaging look into the heart of a computer.