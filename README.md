# Single-Cycle RISC-V RV32I Processor

A simplified **single-cycle** implementation of a **RISC-V (RV32I)** processor core, designed in SystemVerilog.

This project implements the core logic for executing a subset of the RISC-V Instruction Set Architecture (ISA) where every instruction completes its execution in a single clock cycle.

## 🗂️ Project Structure

| File Name | Description |
| :--- | :--- |
| `MCU.sv` | The **Top-Level Module**. Instantiates the CPU, ROM (Instruction Memory), and RAM (Data Memory). |
| `CPU_RV32I.sv` | The **CPU Core**. Connects the Control Unit and Data Path. |
| `DataPath.sv` | Contains the **main data flow components** (Register File, ALU, PC, Adders, Muxes, Imm-Extend). |
| `ControlUnit.sv` | Generates all **control signals** for the data path based on the instruction opcode and function fields. |
| `RAM.sv` | Data Memory module. Supports Byte, Half-word, and Word access (`LB/LH/LW`, `LBU/LHU`, `SB/SH/SW`) with the `strb` signal. |
| `ROM.sv` | Instruction Memory module. Reads the instruction code from `code.mem`. |
| `defines.sv` | SystemVerilog header file containing constants for ALU operations, branch codes, and instruction opcodes. |
| `code.mem` | Hexadecimal file containing the machine code for a test program. |

## ⚙️ Key Features (Single-Cycle Architecture)

* [cite_start]**Architecture:** Single-Cycle.
    * All necessary combinational components (PC, Instruction Fetch, Decode, Execute, Memory Access, Write Back) are connected sequentially without pipeline registers, completing an instruction per clock cycle.
* **Instruction Set:** RISC-V Base Integer Instruction Set (`RV32I`).
* **Instruction Types Implemented:**
    * [cite_start]**R-Type** (Arithmetic/Logic): `ADD`, `SUB`, `SLL`, `SRL`, `SRA`, `SLT`, `SLTU`, `XOR`, `OR`, `AND`[cite: 60, 61, 62, 63, 64, 65].
    * [cite_start]**I-Type** (Immediate/Load): `ADDI`, `SLTI`, `XORI`, `ORI`, `ANDI`, `SLLI`, `SRLI`, `SRAI`, `LB`, `LH`, `LW`, `LBU`, `LHU`[cite: 83, 84, 101, 102].
    * [cite_start]**S-Type** (Store): `SB`, `SH`, `SW`[cite: 84, 98, 99].
    * [cite_start]**B-Type** (Branch): `BEQ`, `BNE`, `BLT`, `BGE`, `BLTU`, `BGEU`[cite: 66, 67, 68].
    * [cite_start]**U-Type** (Upper Immediate): `LUI`, `AUIPC`[cite: 86].
    * [cite_start]**J-Type** (Jump): `JAL`, `JALR`[cite: 87, 88].
* **Memory Access:**
    * [cite_start]Separate Instruction Memory (ROM) and Data Memory (RAM)[cite: 3, 4, 6, 89].
    * [cite_start]The Data RAM supports **byte, half-word, and word** memory operations using a `strb` signal for fine-grained control[cite: 9, 11, 13, 15].

## 💻 Data Path Overview

The `DataPath.sv` module connects the main functional blocks:

1.  [cite_start]**Program Counter (PC):** The `U_PC` register holds the address of the current instruction[cite: 52].
2.  [cite_start]**Instruction Decode & Register File:** Instructions are decoded and read data from the `U_RegFile`[cite: 43].
3.  [cite_start]**ALU:** The `U_ALU` performs arithmetic, logic, and branch comparison operations[cite: 45, 59].
4.  [cite_start]**PC Update:** The `U_PCSrcMux` selects the next PC address based on sequential flow (`PC + 4`), a branch/jump target (`PC_Imm_AdderResult`), or a `JALR` target[cite: 51, 43].
5.  [cite_start]**Write Back:** The `U_RFWDSrcMux` selects the data written back to the register file (`aluResult`, `busRData`, `immExt`, or PC values for `JAL/JALR`)[cite: 46].
