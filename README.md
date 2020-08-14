# EduRISC Instruction Set Architecture (version alpha 0.1)
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
2. Increment `PC`
3. Perform the effects of the instruction as detailed below
  - If the loaded instruction is a `HALT` the processor may halt

## Instruction Set
- [`0x0`: `ADD`](#0x0-add)
- [`0x1`: `SUB`](#0x1-sub)
- [`0x2`: `AND`](#0x2-and)
- [`0x3`: `OR`](#0x3-or)
- [`0x4`: `NAND`](#0x4-nand)
- [`0x5`: `XOR`](#0x5-xor)
- [`0x6`: `SLL` (Shift Left Logical)](#0x5-sll-shift-left-logical)
- [`0x7`: `SRA` (Shift Right Arithmetic)](#0x7-sra-shift-right-arithmetic)
- [`0x8`: `MOVLI` (Move Lower Immediate)](#0x8-movli-move-lower-immediate)
- [`0x9`: `MOVUI` (Move Upper Immediate)](#0x9-movui-move-upper-immediate)
- [`0xA`: `JRN` (Jump Relative if Negative)](#0xa-jrn-jump-relative-if-negative)
- [`0xB`: `JRZ` (Jump Relative if Zero)](#0xb-jrz-jump-relative-if-zero)
- [`0xC`: `JRP` (Jump Relative if Positive)](#0xc-jrp-jump-relative-if-positive)
- [`0xD`: Reserved ](#0xd-reserved)
- [`0xE`: `LOD` (Memory Load)](#0xe-lod-memory-load)
- [`0xF`: `STR` (Memory Store)](#0xf-str-memory-store)

Any data buses that are specified are specifed as left operand, right operand, and output.

### `0x0`: `ADD`
- `0x0XYZ`: `rX = rY + rZ`

### `0x1`: `SUB`
- `0x1XYZ`: `rX = rY - rZ`

### `0x2`: `AND`
- `0x2XYZ`: `rX = rY & rZ`

### `0x3`: `OR`
- `0x3XYZ`: `rX = rY | rZ`

### `0x4`: `NAND`
- `0x4XYZ`: `rX = ~(rY & rZ)`
- Provides NOT if given same operand twice

### `0x5`: `XOR`
- `0x5XYZ`: `rX = rY ^ rZ`

### `0x6`: `SLL` (Shift Left Logical)
- `0x6XYZ`: `rX = rY << rZ`

### `0x7`: `SRA` (Shift Right Arithmetic)
- `0x7XYZ`: `rX = rY >> rZ`
- Sign extends

### `0x8`: `MOVLI` (Move Lower Immediate)
- `0x8XYZ`: `rX = SEXT(0xYZ)`
- Data buses: `0xYZ`, `rX`, `rX`
- Sign extends to fill upper byte of rX

### `0x9`: `MOVUI` (Move Upper Immediate)
- `0x9XYZ`: `rX = 0xYZ00 | (rX & 0x00FF)`
- Data buses: `0xYZ`, `rX`, `rX`
- Leaves lower byte unchanged
- In combination with MOVLI allows setting a register in two instructions

### `0xA`: `JRN` (Jump Relative if Negative)
- `0xAXYZ`: If `rX < 0` then `PC = PC + SEXT(0xYZ)`
- Data buses: `rX`, `PC`, `PC`
- In combination with `JRZ` and `JRP` can produce any condition

### `0xB`: `JRZ` (Jump Relative if Zero)
- `0xBXYZ`: If `rX == 0` then `PC = PC + SEXT(0xYZ)`
- Data buses: `rX`, `PC`, `PC`
- `0xD0FF` effects a `HALT`, repeating this instruction indefinitely
- In combination with `JRN` and `JRP` can produce any condition

### `0xC`: `JRP` (Jump Relative if Positive)
- `0xCXYZ`: If `rX > 0` then `PC = PC + SEXT(0xYZ)`
- Data buses: `rX`, `PC`, `PC`
- In combination with `JRN` and `JRZ` can produce any condition

### `0xD`: Reserved
- Reserved for future use

### `0xE`: `LOD` (Memory Load)
- `0xEXYZ`: `rz = mem[rY]`
- Data buses: `rY`, `rZ`, `rZ`

### `0xF`: `STR` (Memory Store)
- `0xFXYZ`: `mem[rY] = rZ`
- Data buses: `rY`, `rZ`, `rZ`
