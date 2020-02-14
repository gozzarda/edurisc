# EduRISC Instruction Set Architecture
EduRISC is a 16-bit RISC system with a 16-bit address space of 8-bit values for 32 kiB of memory total.

## Registers
EduRISC provides 16 16-bit registers denoted r0 through rF and indexable by a single nibble.
By convention, any read from `r0` will always return 0, regardless of what writes may have been attempted.
Additionally, `rF` is an alias for `PC`, the address in memory of the current instruction.

## Instruction Set
- [`0x0`: `ADD`](#0x0-add)
- [`0x1`: `SUB`](#0x1-sub)
- [`0x2`: `MUL`](#0x2-mul)
- [`0x3`: `AND`](#0x3-and)
- [`0x4`: `OR`](#0x4-or)
- [`0x5`: `NAND`](#0x5-nad)
- [`0x6`: `XOR`](#0x6-xor)
- [`0x7`: `SLL` (Shift Left Logical)](#0x7-sll-shift-left-logical)
- [`0x8`: `SRL` (Shift Right Logical)](#0x8-srl-shift-right-logical)
- [`0x9`: `SRA` (Shift Right Arithmetic)](#0x9-sra-shift-right-arithmetic)
- [`0xA`: `MOVLI` (Move Lower Immediate)](#0xa-movli-move-lower-immediate)
- [`0xB`: `MOVUI` (Move Upper Immediate)](#0xb-movui-move-upper-immediate)
- [`0xC`: `JRI` (Jump Relative Immediate)](#0xc-jri-jump-relative-immediate)
- [`0xD`: `JAC` (Jump Absolute Conditional)](#0xd-jac-jump-absolute-conditional)
- [`0xE`: `MEM` (Memory Access)](#0xe-mem-memory-access)
- [`0xF`: `SYS` (System Call)](#0xf-sys-system-call)

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
- `0xAXYZ`: `rX = 0x00YZ`
- Data buses: `0xYZ`, `rX`, `rX`
- Zeros out upper byte of rX

### `0xB`: `MOVUI` (Move Upper Immediate)
- `0xBXYZ`: `rX = 0xYZ00 | (rX & 0x00FF)`
- Data buses: `0xYZ`, `rX`, `rX`
- Leaves lower byte unchanged
- In combination with MOVLI allows setting a register in two instructions

### `0xC`: `JRI` (Jump Relative Immediate)
- `0xCXYZ`: `PC = PC + SEXT(0xXYZ) << 1`
- Data buses: `SEXT(0xXYZ)`, `PC`, `PC`
- Sign extends the lower three nibbles of the instruciton and adds that number of words to the program counter

### `0xD`: `JAC` (Jump Absolute Conditional)
- `0xDXYZ`: `PC = rZ` (conditional)
- Data buses: `rY`, `rZ`, `PC`
- Where `X` is a 4-bit flag set `0bNZP0`
- `PC` is only updated if `(N && rY < 0) || (Z && rY == 0) || (P && rY > 0)`
- Tests `rZ` relative to 0 and jumps if condition is met

### `0xE`: `MEM` (Memory Access)
- `0xEXYZ`
- Data buses: `rY`, `rZ`, `rZ`
- Where `X` is a 4-bit flag set `0bSLE0`:
	- If `S` is 0, the operation is a `LOAD`: `rZ = mem[rY] & (mem[rY+1] << 8)`
		- If `L` is set, load only the lower byte: `rZ = mem[rY]`
			- If `E` is set, sign-extend: `rZ = mem[rY] + mem[rY][7] * 0xFF00`
	- If `S` is 1, the operation is a `STORE`: `mem[rY] = rZ[0..7]`, `mem[rY+1] = rZ[8..15]`
		- If `L` is set, store only the lower byte: `mem[rY] = rZ[0..7]`
			- If `E` is set, sign-extend: `mem[rY] = rZ[0..7]`, `mem[rY+1] = rZ[7] * 0xFF`

### `0xF`: `SYS` (System Call)
- `0xFXYZ`: Perform system call with id `0xYZ`, providing it a access to `rX`.
- Data buses: `0xYZ`, `rX`, `rX`
- This allows for up to 256 system calls
- Currently this specification proposes four:
	- `0xFX00`: Suspend until next event
		- Event is loosely defined as any influence that could free the system from its suspended state, such as IO or timers
	- `0xFX01`: Put Byte
		- Send the value stored in the lower byte of `X` out over our output stream
	- `0xFX02`: Can Read
		- Sets `rX` to `0` if there is no input waiting on our input stream, or some non-zero value if there is
	- `0xFX03`: Read Byte
		- Sets `rX` to the next value in the input stream and progresses the input stream by one byte
