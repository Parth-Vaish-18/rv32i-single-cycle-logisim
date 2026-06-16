# RISC-V (RV32I Base Integer ISA) Microprocessor in Logisim

A single-cycle RV32I RISC-V processor implemented visually in Logisim Evolution 4.1.0. The design is fully hierarchical, with each functional unit built as its own sub-circuit and wired together in a top-level `main` datapath.

---

## Overview

This project implements a **single-cycle CPU** for the RV32I base integer instruction set. Every instruction completes in one clock cycle: the program counter advances, the instruction is fetched from ROM, decoded, executed by the ALU, and the result is written back — all within a single rising edge. The design is laid out as interconnected Logisim sub-circuits, making each stage independently inspectable and testable.

---

## Sub-Circuits

| Circuit | Description |
|---|---|
| `main` | Top-level datapath connecting all units |
| `Program_Counter` | 32-bit PC register with synchronous reset and `PC_in` mux |
| `PC_plus4` | Adds 4 to the current PC for sequential fetch |
| `Control` | Main control unit — decodes opcode + funct3 and generates all datapath control signals |
| `ALU_Control` | Derives the 4-bit `ALUSel` signal from opcode, funct3, and funct7 |
| `ALU` | 32-bit arithmetic/logic unit, operation selected by `ALUSel` |
| `RegFile` | 32×32-bit register file; dual read ports (rs1, rs2), single write port (rd) with write-enable |
| `Imm_Gen` | Immediate generator — sign-extends and reconstructs all five immediate formats |
| `Branch_Comp` | Compares rs1 and rs2 for BEQ, BLT, and BLTU; feeds branch outcome back to Control |
| `Data_Mem` | Data memory with byte/halfword/word access controlled by funct3 and write-enable |
| `i_splitter` | Extracts fields for I-type instructions (rd, funct3, rs1, imm[11:0], opcode) |
| `s_splitter` | Extracts fields for S-type instructions (rs1, rs2, imm[4:0], imm[11:5], funct3, opcode) |
| `b_splitter` | Extracts fields for B-type instructions (rs1, rs2, imm bits 1–4, 5–10, 11, 12) |
| `j_splitter` | Extracts fields for J-type instructions (rd, imm bits 1–10, 11, 12–19, 20, opcode) |
| `u_splitter` | Extracts fields for U-type instructions (rd, imm[31:12], opcode) |

---

## Control Signals

The `Control` unit drives the following signals based on opcode and funct3:

| Signal | Description |
|---|---|
| `reg_write_sel` | Selects what is written back to the register file (ALU result, memory data, PC+4, immediate) |
| `imm_tk` | Chooses immediate vs register as ALU B-operand |
| `mem_we` | Data memory write enable |
| `reg_we` | Register file write enable |
| `pc_tk` | Selects next PC source (PC+4 vs branch/jump target) |
| `branch_tk` | Enables branch target when branch condition is met |

Branch resolution is combinational: `Branch_Comp` computes BEQ, BLT, and BLTU in parallel and feeds the results back to `Control`, which gates `branch_tk` accordingly.

---

## Immediate Formats

All five RV32I immediate encodings are decoded by dedicated splitter sub-circuits that extract the correct instruction fields and route them to `Imm_Gen` for sign-extension and reconstruction:

- **I-type** — ADDI, loads, JALR
- **S-type** — stores
- **B-type** — branches
- **J-type** — JAL
- **U-type** — LUI, AUIPC

---

## Requirements

- [Logisim Evolution v4.1.0](https://github.com/logisim-evolution/logisim-evolution/releases)

---

## Usage

1. Download and launch Logisim Evolution 4.1.0.
2. Open `logisim.circ` via **File → Open**.
3. Navigate to the `main` circuit in the left panel to see the full datapath.
4. Load a RV32I program into the ROM component (right-click → **Edit Contents**).
5. Use **Simulate → Tick Clock** (or enable auto-tick) to step through execution.
6. Probe any wire with the **Poke Tool** to inspect signal values in real time.

---

## Design Notes

- The register file enforces `x0 = 0` by ignoring writes when `rdesi = 0`.
- Data memory supports byte (`funct3 = 000/100`), halfword (`001/101`), and word (`010`) granularity, matching LB/LH/LBU/LHU/LW and SB/SH/SW semantics.
- The Branch Comparator performs both signed (BLT/BGE) and unsigned (BLTU/BGEU) comparisons combinationally.
- All sub-circuits are self-contained and can be simulated independently for unit-level debugging.

---
