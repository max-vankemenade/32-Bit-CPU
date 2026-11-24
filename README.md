# 32-Bit RISC-V CPU in Digital Logic Sim

![RISC-V Architecture](https://img.shields.io/badge/Architecture-RISC--V%20(RV32I)-blue)
![Simulator](https://img.shields.io/badge/Simulator-Sebastian%20Lague's%20Digital%20Logic%20Sim-orange)

A fully functional, programmable 32-bit Central Processing Unit (CPU) designed and wired from scratch at the gate level using Sebastian Lague’s **[Digital Logic Sim](https://sebastian.itch.io/digital-logic-sim)**. Built around the **RISC-V (RV32I)** Instruction Set Architecture, this project demonstrates low-level computer architecture, digital logic design, and hardware-software co-design.

<img width="1678" height="968" alt="image" src="https://github.com/user-attachments/assets/8a12505e-2f73-4737-9dc5-9b9cd3afa1d1" />

---

## Project Motivation & Goals
The objective of this project was to bridge the gap between theoretical computer architecture and gate-level implementation starting with just the single built-in `NAND` gate. Rather than using high-level Hardware Description Languages (HDLs) like Verilog or VHDL, every multiplexer, ALU slice, register, and control signal was manually routed using fundamental logic gates (`AND`, `OR`, `NOT`, `XOR`) along with MS flip-flops and tri-state buffers to gain a practical understanding of:
-   Datapath design and control signal orchestration.
-   Instruction fetching, decoding, and execution cycles.
-   Binary arithmetic, two's complement operations, and bitwise logic at the gate abstraction level.
-   Program execution flow (branching and jumping).

---

## System Architecture

### Core Specifications
-   **ISA:** RISC-V (RV32I Base Integer Instruction Set subset)
-   **Data Width:** 32-bit datapath and registers
-   **Execution Model:** Single-cycle
-   **Register File:** 16 × 32-bit general-purpose registers (`x0` hardwired to `0x00000000`)
-   **Instruction Memory (SRC):** 256 × 32-bit words
-   **Data Memory (RAM):** 256 bytes
-   **Clock Speed:** Manual stepping or configurable auto-clock in Digital Logic Sim (`Steps per clock tick` = 6, `Steps per second (target)` = 60)

### High-Level Datapath Overview
```text
+-----------------+     +-----------------+     +------------------------------------+
| Program Counter | --> |   Instruction   | --> |            CONTROL UNIT            |
|      (PC)       |     |      (SRC)      |     | (Decodes Opcode, Funct3, Funct7 -> |
+-----------------+     +-----------------+     |   ALU Op, RegWrite, MemRW, etc.)   |
         ^                                      +------------------------------------+
         |                                         |         |         |          |
         |        +--------------------------------+         |         |          |
         |        |             +----------------------------+         |          |
         |        |             |                  +-------------------+          |
         |        v             v                  v                              v
+-----------------+     +-----------------+     +-----------------+     +-----------------+
|     Branch      | <-- |   32-Bit ALU    | <-- |    Registers    | <-> |   Data Memory   |
|      Logic      |     | (+, -, &, |,...)|     |  (16 x 32-bit)  |     |      (RAM)      |
+-----------------+     +-----------------+     +-----------------+     +-----------------+
```

---

## ALU

<img width="1798" height="625" alt="image" src="https://github.com/user-attachments/assets/433badd9-76ca-4cd7-b99e-dc1b0dc698b6" />

Decodes the 4-bit `ALU Control` signal sent by the Control Unit to select the appropriate component for computation.

## Control Unit
<img width="839" height="937" alt="image" src="https://github.com/user-attachments/assets/fa8fd530-6299-4502-9a4d-cbed45a21765" />

Decodes the 32-bit instruction word from the `SRC` into the necessary control signals (`ALUSrc`, `MemtoReg`, `RegWrite`, `MemRead`, `MemWrite`, `Branch`, `Jump`, etc.). `IN-Slicer` splits the instruction apart into `op`, `funct3`, `funct7`, `rd`, `rs1`, and `rs2`. Signal control sets control signals and ALU operations with logic gates, while `CU IMM` reconstructs the immediate value based on `op`.

## Register File
<img width="575" height="928" alt="image" src="https://github.com/user-attachments/assets/4f08e0f4-ef4f-48e2-8ee3-74e7f722f5f4" />

16 × 32-bit registers addressed via the 4-bit `rd`, `rs1`, and `rs2` outputs of the Control Unit. Writes are synchronized to the falling clock edge. Register `Rg0` (`x0`) ignores write inputs and always evaluates to `0`.

## Coding (`SRC`)
<img width="1591" height="905" alt="image" src="https://github.com/user-attachments/assets/14edc02b-42d0-4680-b7bf-4d5c7447b885" />

The image shows one of four code blocks, each with a 6-bit input that decodes to 64 individual lines of code (4 × 64 = 256 lines). To write code, connect a wire from one of the decoder output lines to your desired instruction inputs (`R-Type`, `I-Type`, `L-Type`, etc.), which outputs the corresponding `op`, `funct3`, and `funct7`. For operands, route wires to registers or manually wire an immediate value into the `IMM` chip. When the decoder outputs `1`, `OP-ENC` encodes the raw instruction into a standard RV32I 32-bit instruction word. In this example, the active line is:

```assembly
slli x6, x6, 1
```

Which encodes to `0x00131313`.

---

## Hardware Components Built From Scratch

To keep the circuit modular and maintainable inside Digital Logic Sim, the CPU was constructed hierarchically using custom sub-circuits (chips):

1.  **32-Bit Arithmetic Logic Unit (ALU):**
    -   Supports addition, subtraction (via two's complement), bitwise `AND`, `OR`, `XOR`, logical/arithmetic shifts (`SLL`, `SRL`, `SRA`), and set-less-than comparisons (`SLT`, `SLTU`).
    -   Outputs are also used to drive conditional branching.
2.  **16 × 32-Bit Register File:**
    -   Dual read ports (`rs1`, `rs2`) and a single write port (`rd`) enabled on the falling clock edge.
    -   Enforces the RISC-V rule where register `x0` always evaluates to `0` regardless of write attempts.
3.  **Immediate Generator (ImmGen):**
    -   Extracts and sign-extends 12-bit and 20-bit immediates across `I-Type`, `S-Type`, `B-Type`, `U-Type`, and `J-Type` instruction formats into full 32-bit values.
4.  **Control Unit & ALU Decoder:**
    -   Combinational logic network translating the 7-bit `opcode`, 3-bit `funct3`, and 7-bit `funct7` fields into control signals (`ALUSrc`, `MemtoReg`, `RegWrite`, `MemRead`, `MemWrite`, `Branch`, `Jump`, etc.).
5.  **Program Counter (PC) & Branch Logic:**
    -   Handles sequential execution (`PC + 1`), conditional branch offsets (`BEQ`, `BNE`, `BLT`, `BGE`, etc.), and unconditional jumps (`JAL`, `JALR`).

---

## Supported Instruction Set (RV32I)

| Format | Instructions Supported | Description |
| :--- | :--- | :--- |
| **R-Type** | `ADD`, `SUB`, `AND`, `OR`, `XOR`, `SLL`, `SRL`, `SRA`, `SLT`, `SLTU` | Register-to-register arithmetic and logic |
| **I-Type** | `ADDI`, `ANDI`, `ORI`, `XORI`, `SLTI`, `SLTIU`, `SLLI`, `SRLI`, `SRAI` | Immediate arithmetic and logic |
| **Load** | `LW`, `LH`, `LB`, `LHU`, `LBU` | Load data from memory into registers |
| **Store** | `SW`, `SH`, `SB` | Store register data into memory |
| **Branch** | `BEQ`, `BNE`, `BLT`, `BGE`, `BLTU`, `BGEU` | Conditional PC-relative branching |
| **Jump** | `JAL`, `JALR` | Unconditional jumps and link register saving |
| **Upper** | `LUI`, `AUIPC` | Load or add upper 20-bit immediates |

---

## Example Program: Fibonacci Sequence

To verify full programmability—including loops, conditional branches, register manipulation, and memory storage—the CPU was programmed to compute the Fibonacci sequence.

### RISC-V Assembly
```assembly
# Compute the first 64 prime numbers and store them in RAM[0..252]
    and  x10, x0, x0        # x10 = 0 (RAM write pointer)
    addi x12, x0, 3         # x12 = 3 (first odd prime candidate)
    addi x1,  x0, 2         # x1  = 2 (first prime)
    addi x15, x0, 252       # x15 = 252 (target RAM byte offset: 64 words)
    sw   x1,  0(x10)        # RAM[0] = 2

cand_loop:
    addi x11, x0, 4         # x11 = 4 (divisor pointer, starting at RAM[4] = 3)

div_loop:
    blt  x10, x11, is_prime # No stored divisors left -> candidate is prime
    lw   x2,  0(x11)        # x2 = prime divisor from RAM
    addi x5,  x0, 0         # x5 = 0 (accumulator for x2^2)
    add  x6,  x2, x0        # x6 = x2 (multiplicand)
    add  x7,  x2, x0        # x7 = x2 (multiplier / shift counter)

sq_loop:                    # Compute x5 = x2 * x2 via shift-and-add
    andi x8,  x7, 1         # Check LSB of x7
    beq  x8,  x0, sq_skip   # Skip add if LSB == 0
    add  x5,  x5, x6        # x5 += x6
sq_skip:
    slli x6,  x6, 1         # x6 <<= 1
    srli x7,  x7, 1         # x7 >>= 1
    bne  x7,  x0, sq_loop   # Repeat until x7 == 0

    blt  x12, x5, is_prime  # If candidate < divisor^2 -> candidate is prime
    add  x1,  x12, x0       # x1 = candidate (dividend)

rem_loop:                   # Compute x1 = candidate mod divisor
    sub  x1,  x1, x2        # x1 -= x2
    bge  x1,  x2, rem_loop  # Repeat while remainder >= divisor

    beq  x1,  x0, not_prime # Remainder == 0 -> divisible -> not prime
    addi x11, x11, 4        # Advance to next prime divisor in RAM
    jal  x0,  div_loop      # Test next divisor

is_prime:
    addi x10, x10, 4        # Advance RAM write pointer
    sw   x12, 0(x10)        # Store discovered prime in RAM
    beq  x10, x15, end      # Halt once RAM[252] is filled

not_prime:
    addi x12, x12, 2        # Next odd candidate (+2)
    jal  x0,  cand_loop     # Evaluate next candidate

end:
    jal  x0,  end           # Infinite loop to halt execution
```

## How to Run & Test

1.  **Download Digital Logic Sim:** Install Sebastian Lague's [Digital Logic Sim](https://sebastian.itch.io/digital-logic-sim).
2.  **Clone this Repository:**
    ```bash
    git clone https://github.com/max-vankemenade/32-Bit-CPU.git
    ```
3.  **Load the Project:**
    -   Copy the project folder into Digital Logic Sim's save directory:
        -   **Windows:** `%USERPROFILE%\AppData\LocalLow\SebastianLague\Digital Logic Sim\Projects`
4.  **Execute a Program:**
    -   Open the `CPU` chip in the simulator.
    -   Under **Menu > Prefs > Simulation**, set `Steps per clock tick` = 6 and `Steps per second (target)` = 60.
    -   Toggle the green `Reset` input on and off so the PC outputs `255`.
    -   Toggle the orange `Step` input on and off so the PC outputs `0`.
    -   Turn on the green `Run` input.

## License
This project is open-source and available under the [MIT License](LICENSE).
