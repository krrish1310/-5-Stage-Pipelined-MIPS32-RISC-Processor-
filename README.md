# Pipelined MIPS-32 Processor (`pipe_MIPS32`)

## Overview

This project implements a **5-stage pipelined MIPS-32 processor** in Verilog. The processor supports a simplified instruction set and handles basic hazards like control hazards using branch flags. The processor uses a **two-phase clock** (`clk1` and `clk2`) to simulate overlapped pipeline stages.

## Pipeline Stages

The processor consists of the following classic 5 stages:

1. **IF (Instruction Fetch)**  
   - Fetches instruction from memory
   - Updates the program counter (PC)
   - Branch instructions evaluated here using earlier stage outputs

2. **ID (Instruction Decode & Register Fetch)**  
   - Decodes the instruction
   - Fetches source operands from the register file
   - Sign-extends immediate values

3. **EX (Execute / Address Calculation)**  
   - Performs ALU operations or computes branch target address
   - Evaluates branch condition flags

4. **MEM (Memory Access)**  
   - Loads or stores data to/from memory
   - For branch instructions, instructions after the taken branch are flushed

5. **WB (Write Back)**  
   - Writes the result to the register file (for ALU or Load)

## Instruction Set

### ALU Instructions
| Instruction | Opcode   | Type   | Operation             |
|-------------|----------|--------|------------------------|
| `ADD`       | 000000   | RR_ALU | `rd = rs + rt`         |
| `SUB`       | 000001   | RR_ALU | `rd = rs - rt`         |
| `AND`       | 000010   | RR_ALU | `rd = rs & rt`         |
| `OR`        | 000011   | RR_ALU | `rd = rs | rt`         |
| `SLT`       | 000100   | RR_ALU | `rd = (rs < rt)`       |
| `MUL`       | 000101   | RR_ALU | `rd = rs * rt`         |

### Immediate ALU Instructions
| Instruction | Opcode   | Type   | Operation               |
|-------------|----------|--------|--------------------------|
| `ADDI`      | 001000   | RM_ALU | `rt = rs + imm`          |
| `SUBI`      | 001001   | RM_ALU | `rt = rs - imm`          |
| `SLTI`      | 001100   | RM_ALU | `rt = (rs < imm)`        |

### Memory Instructions
| Instruction | Opcode   | Type   | Operation                 |
|-------------|----------|--------|----------------------------|
| `LW`        | 100000   | LOAD   | `rt = Mem[rs + imm]`       |
| `SW`        | 100001   | STORE  | `Mem[rs + imm] = rt`       |

### Branch Instructions
| Instruction | Opcode   | Type   | Operation                             |
|-------------|----------|--------|----------------------------------------|
| `BEQZ`      | 001110   | BRANCH | `if (rs == 0) PC += offset`            |
| `BNEQZ`     | 001101   | BRANCH | `if (rs != 0) PC += offset`            |

### Halt
| Instruction | Opcode   | Type  | Operation          |
|-------------|----------|-------|---------------------|
| `HLT`       | 111111   | HALT | Stop execution       |

## Registers & Memory

- **Registers**: 32 general-purpose registers (`Reg[0]` to `Reg[31]`)
- **Memory**: 1024-word memory (`Mem[0]` to `Mem[1023]`)
- Immediate values are sign-extended before use.

## Control Flags

- `HALTED`: Set to 1 when `HLT` instruction is executed.
- `TAKEN_BRANCH`: Flag used to flush instructions after a branch is taken.

## Clocks

- `clk1`: Drives IF, EX, WB stages
- `clk2`: Drives ID, MEM stages
- The two-phase clocking helps mimic hardware overlap of pipeline stages.

## Hazards Handling

- **Control Hazard**: Branch decision is made in the EX stage. The `TAKEN_BRANCH` flag disables write-back or store for instructions in the wrong path.
- **Data Hazards**: **Not handled** explicitly (no forwarding or stall logic implemented). This is a basic implementation focusing on pipelining structure.

## Usage

### Simulation

1. Write a testbench to load memory (`Mem[]`) with instructions and initial data.
2. Initialize `PC = 0`, `HALTED = 0`.
3. Run simulation with alternating clocks (`clk1`, `clk2`).
4. Monitor `Reg[]`, `Mem[]`, and `HALTED` to check results.

### Sample Verilog Testbench Snippet

```verilog
initial begin
  clk1 = 0; clk2 = 0;
  repeat (50) begin
    #5 clk1 = 1; #5 clk1 = 0;
    #5 clk2 = 1; #5 clk2 = 0;
  end
end
