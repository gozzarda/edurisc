# EduRISC Instruction Set Architecture
EduRISC is a RISC system intended for teaching computer theory.
EduRISC is a 16-bit RISC system with a 16-bit address space of 16-bit words for 32,768 words (64 kiB) of total addressable space.
Priority is given to ease of understanding while remaining sufficiently practical to give a sense of how real processors operate.

## I/O
I/O is performed through memory-mapped I/O devices.
The exact specification and memory location of these devices is beyond the scope of this document.

## Registers
EduRISC provides 16 16-bit registers denoted `r0` through `rF` and indexable by a single nibble.
By convention, any read from `r0` will always return 0, regardless of what writes may have been attempted.
Additionally, `rF` is an alias for `PC`, the address in memory of the current instruction.

## Data Buses
In abstract, an EduRISC procressor can be thought of as having four data buses: Instruction, Left Operand, Right Operand, and Output.
The Instruction, Left, and Right buses contain all the processor-internal information that may be used to complete an instruction.
The Output bus provides the only method for the instruction modifying the processor's internal state.

## Operation
Typical operation of an EduRISC machine repeats the following steps:
1. Load the current instruction from memory at the address stored in `PC`
  - If the loaded instruction is a `HALT` the processor may halt
2. Perform the effects of the instruction as detailed below
3. Increment `PC`

## Instruction Set
- [`0x0`: `ADD`](#0x0-add)
- [`0x1`: `SUB`](#0x1-sub)
- [`0x2`: `MUL`](#0x2-mul)
- [`0x3`: `AND`](#0x3-and)
- [`0x4`: `OR`](#0x4-or)
- [`0x5`: `NAND`](#0x5-nand)
- [`0x6`: `XOR`](#0x6-xor)
- [`0x7`: `SLL` (Shift Left Logical)](#0x7-sll-shift-left-logical)
- [`0x8`: `SRL` (Shift Right Logical)](#0x8-srl-shift-right-logical)
- [`0x9`: `SRA` (Shift Right Arithmetic)](#0x9-sra-shift-right-arithmetic)
- [`0xA`: `MOVLI` (Move Lower Immediate)](#0xa-movli-move-lower-immediate)
- [`0xB`: `MOVUI` (Move Upper Immediate)](#0xb-movui-move-upper-immediate)
- [`0xC`: `JRN` (Jump Relative if Negative)](#0xc-jrn-jump-relative-if-negative)
- [`0xD`: `JRZ` (Jump Relative if Zero)](#0xd-jrz-jump-relative-if-zero)
- [`0xE`: `JRP` (Jump Relative if Positive)](#0xe-jrp-jump-relative-if-positive)
- [`0xF`: `MEM` (Memory Access)](#0xf-mem-memory-access)

Any data buses that are specified are specifed as left operand, right operand, and output.

### `0x0`: `ADD`
- `0x0XYZ`: `rX = rY + rZ`

### `0x1`: `SUB`
- `0x1XYZ`: `rX = rY - rZ`

### `0x2`: `MUL`
- `0x2XYZ`: `rX = rY * rZ`

### `0x3`: `AND`
- `0x3XYZ`: `rX = rY & rZ`

### `0x4`: `OR`
- `0x4XYZ`: `rX = rY | rZ`

### `0x5`: `NAND`
- `0x5XYZ`: `rX = ~(rY & rZ)`
- Provides NOT if given same operand twice

### `0x6`: `XOR`
- `0x6XYZ`: `rX = rY ^ rZ`

### `0x7`: `SLL` (Shift Left Logical)
- `0x7XYZ`: `rX = rY << rZ`

### `0x8`: `SRL` (Shift Right Logical)
- `0x8XYZ`: `rX = rY >> rZ`

### `0x9`: `SRA` (Shift Right Arithmetic)
- `0x9XYZ`: `rX = rY >> rZ`
- Sign extends

### `0xA`: `MOVLI` (Move Lower Immediate)
- `0xAXYZ`: `rX = SEXT(0xYZ)`
- Data buses: `0xYZ`, `rX`, `rX`
- Sign extends to fill upper byte of rX

### `0xB`: `MOVUI` (Move Upper Immediate)
- `0xBXYZ`: `rX = 0xYZ00 | (rX & 0x00FF)`
- Data buses: `0xYZ`, `rX`, `rX`
- Leaves lower byte unchanged
- In combination with MOVLI allows setting a register in two instructions

### `0xC`: `JRN` (Jump Relative if Negative)
- `0xCXYZ`: If `rX < 0` then `PC = PC + SEXT(0xYZ)`
- Data buses: `rX`, `PC`, `PC`
- In combination with `JRZ` and `JRP` can produce any condition

### `0xD`: `JRZ` (Jump Relative if Zero)
- `0xDXYZ`: If `rX == 0` then `PC = PC + SEXT(0xYZ)`
- Data buses: `rX`, `PC`, `PC`
- `0xD0FF` effects a `HALT`, repeating this instruction indefinitely
- In combination with `JRN` and `JRP` can produce any condition

### `0xE`: `JRP` (Jump Relative if Positive)
- `0xEXYZ`: If `rX > 0` then `PC = PC + SEXT(0xYZ)`
- Data buses: `rX`, `PC`, `PC`
- In combination with `JRN` and `JRZ` can produce any condition
			
### `0xF`: `MEM` (Memory Access)
- `0xFXYZ`
- Data buses: `rY`, `rZ`, `rZ`
- `0xF0YZ` is a `LOAD`: `rZ = mem[rY]`
- `0xF8YZ` is a `STORE`: `mem[rY] = rZ`
