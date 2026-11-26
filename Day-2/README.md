# Day 2 – Application Binary Interface (ABI)

## 1. Introduction to ABI

**ABI (Application Binary Interface)** defines the rules that allow compiled programs to communicate with the operating system and processor.

It specifies:
- Register usage and naming conventions.
- Function calling conventions.
- Argument passing and return values.
- Stack pointer and memory allocation.
- Data sizes and alignment.
- System call conventions.

### RV64 Architecture
- XLEN = 64 bits.
- 32 general-purpose registers (`x0` to `x31`).
- Each register is 64 bits wide.
- Base integer instruction set: RV64I.

---

## 2. ABI – Layered Architecture
![image]()

The ABI acts as a contract between software and hardware.

### Layers (Top to Bottom)

1. **Application Program:** Programs such as email applications.
2. **API (Application Programming Interface):** Source-level interfaces used by programmers.
   - Java: `java.text`
   - C: `stdio.h`
   - C++: `std::array`
3. **Standard Libraries:** Provide functions built on top of operating system services.
4. **System Calls:** Allow applications to request services from the operating system, such as file operations and memory management.
5. **Operating System:** Examples include Linux, Windows and macOS.
6. **ISA (Instruction Set Architecture):** Defines processor instructions, such as RISC-V, ARM and x86.
7. **RTL and Hardware:** Hardware implementation of the processor architecture.

### Role of ABI

ABI connects compiled software with the operating system and processor.

It defines:
- Calling conventions.
- Register usage.
- Data sizes and alignment.
- System call conventions.
- Executable and library binary formats.

### Example: Disassembled RISC-V Main Function

```asm
addi sp, sp, -80
sd   s7, 8(sp)
lui  s7, 0x0
mv   a0, s7
sd   ra, 72(sp)
...
jalr ra
```

**Explanation:**
- `addi sp, sp, -80` – Allocates 80 bytes of stack space.
- `sd s7, 8(sp)` – Saves the value of s7 in memory.
- `lui s7, 0x0` – Loads an upper immediate value.
- `mv a0, s7` – Copies the value of s7 into a0.
- `sd ra, 72(sp)` – Saves the return address.
- `jalr` – Performs an indirect jump and link.

---

## 3. ABI Register Convention

RISC-V has 32 general-purpose registers, from `x0` to `x31`.

Each register has:
- Register number.
- ABI name.
- Specific purpose.
- Caller-saved or callee-saved convention.

### Register Table

| Register | ABI Name | Usage | Saver |
|---|---|---|---|
| x0 | zero | Hard-wired zero | - |
| x1 | ra | Return address | Caller |
| x2 | sp | Stack pointer | Callee |
| x3 | gp | Global pointer | - |
| x4 | tp | Thread pointer | - |
| x5–x7 | t0–t2 | Temporary registers | Caller |
| x8 | s0/fp | Saved register / frame pointer | Callee |
| x9 | s1 | Saved register | Callee |
| x10–x11 | a0–a1 | Function arguments / return values | Caller |
| x12–x17 | a2–a7 | Function arguments | Caller |
| x18–x27 | s2–s11 | Saved registers | Callee |
| x28–x31 | t3–t6 | Temporary registers | Caller |

**Note:** The table above follows the slide's register naming and usage, with the standard RISC-V caller/callee preservation convention. Temporary registers t3–t6 are caller-saved.

### Important Register Groups

- `zero` – Always contains zero.
- `ra` – Stores the return address.
- `sp` – Points to the stack.
- `gp` – Global pointer.
- `tp` – Thread pointer.
- `t0–t6` – Temporary registers.
- `s0–s11` – Saved registers.
- `a0–a7` – Function argument registers.
- `a0–a1` – Also used for return values.

### Caller-Saved vs Callee-Saved

**Caller-Saved:**
- The calling function must save the register if it needs the value after a function call.
- Examples: `t0–t6`, `a0–a7`, `ra`.

**Callee-Saved:**
- The called function must restore the original register value before returning if it modifies that register.
- Examples: `s0–s11`, `sp`.

---

## 4. Memory Allocation and Stack Pointer

The stack is a memory area used during function calls.

### Stack Pointer (sp)

- `sp` is register `x2`.
- It stores the current stack address.
- It is used for allocating memory during function execution.
- Local variables and saved registers can be stored on the stack.

### Memory Addressing

- Memory is byte-addressable.
- One byte = 8 bits.
- One doubleword = 8 bytes = 64 bits.

Example:

| Doubleword | Memory Address |
|---|---|
| 1st | M[0] |
| 2nd | M[8] |
| 3rd | M[16] |
| 4th | M[24] |

### Little-Endian Format

RISC-V systems commonly use little-endian memory representation.

- Least significant byte is stored at the lowest memory address.
- Most significant byte is stored at the highest memory address.

---

## 5. Load Word Instruction (LD)

### Example

```asm
ld x8, 16(x23)
```

**Purpose:** Loads a 64-bit doubleword from memory into a register.

### Instruction Breakdown

| Field | Value | Meaning |
|---|---|---|
| rd | x8 | Destination register |
| rs1 | x23 | Base address register |
| Immediate | 16 | Offset |
| Instruction | ld | Load doubleword |

### Operation

```text
Effective Address = x23 + 16
x8 = Memory[Effective Address]
```

If `x23 = 0`:

```text
Effective Address = 0 + 16 = 16
```

The processor reads 8 bytes from addresses M[16] to M[23].

### Example Memory Data

Bytes from lowest to highest address:

```text
10111111
10111101
10001100
01011000
01001100
01001001
00011111
01110010
```

Loaded 64-bit value:

```text
0x721F494C588CBD BF
```

Correct continuous hexadecimal representation:

```text
0x721F494C588CBDBF
```
## 6. Store Word Instruction (SW)

### Example

```asm
sw x8, 8(x23)
```

**Purpose:** Stores a 32-bit word (4 bytes) from a register into memory.

### Instruction Breakdown

| Field | Value | Meaning |
|---|---|---|
| rs2 | x8 | Source/data register |
| rs1 | x23 | Base address register |
| Immediate | 8 | Offset |
| Instruction | sw | Store word |

### Operation

```text
Effective Address = x23 + 8
Memory[Effective Address] = x8[31:0]
```

If `x23 = 0`:

```text
Effective Address = 0 + 8 = 8
```

The processor stores 4 bytes from the lower 32 bits of x8 into memory addresses:

```text
M[8], M[9], M[10], M[11]
```

### Example

Assume:

```text
x8  = 0x123456789ABCDEF0
x23 = 0
```

The `sw` instruction stores only the lower 32 bits:

```text
0x9ABCDEF0
```

Because RISC-V uses little-endian memory representation:

| Memory Address | Stored Byte |
|---|---|
| M[8] | F0 |
| M[9] | DE |
| M[10] | BC |
| M[11] | 9A |


---

## 8. I-Type, R-Type and S-Type Formats

RISC-V uses different instruction formats depending on the operation.

### I-Type
Used for immediate operations and load instructions.

Example:

```asm
ld x8, 16(x23)
```

### R-Type
Used for register-to-register arithmetic and logical operations.

Example:

```asm
add x8, x24, x8
```

### S-Type
Used for store instructions.

Example:

```asm
sd x8, 8(x23)
```



---

## 9. Example

The three instructions work together to load data, perform addition and store the result.

```asm
ld  x8, 16(x23)    # Load data from memory
add x8, x24, x8    # Add register values
sd  x8, 8(x23)     # Store result in memory
```

### Step-by-Step Execution

**Step 1: Load**

```asm
ld x8, 16(x23)
```

Loads a doubleword from memory address `x23 + 16` into x8.

**Step 2: Addition**

```asm
add x8, x24, x8
```

Adds x24 and x8. The result is stored in x8.

**Step 3: Store**

```asm
sd x8, 8(x23)
```

Stores the result in memory address `x23 + 8`.

### Load and Store Comparison

| Feature | LD (Load) | SD (Store) |
|---|---|---|
| Direction | Memory → Register | Register → Memory |
| Format | I-Type | S-Type |
| Data size | 8 bytes | 8 bytes |
| Destination | rd | Memory |
| Base register | rs1 | rs1 |
| Offset | Immediate | Immediate |

---

## LAB 
![image]()


