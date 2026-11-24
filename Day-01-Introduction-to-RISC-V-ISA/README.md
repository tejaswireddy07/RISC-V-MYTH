# Day 1 – Introduction to RISC-V Architecture and GNU Compiler Toolchain

## 1. Introduction

RISC-V is an open-standard Instruction Set Architecture (ISA) based on Reduced Instruction Set Computer (RISC) principles. It provides a simple, modular and extensible architecture that allows developers to design and implement processors for different applications.

Day 1 focuses on understanding the fundamentals of RISC-V architecture and the working of the GNU Compiler Toolchain. It introduces the complete software-to-hardware execution flow, starting from writing a C program and converting it into machine-level instructions that a RISC-V processor can execute.

The practical activities cover compiling C programs using GCC, cross-compiling them using the RISC-V GNU Compiler Toolchain, generating object files, examining RISC-V assembly instructions using Objdump and executing programs through the Spike instruction set simulator.

These exercises provide practical knowledge of how high-level programming languages are translated into RISC-V machine instructions. They also help in understanding compiler optimization, instruction execution, processor registers and basic debugging concepts.

By the end of Day 1, the practical work establishes a foundation for understanding RISC-V instruction execution, compiler operations and processor architecture.
# 2. Types of RISC-V Instructions

RISC-V follows a modular Instruction Set Architecture (ISA). It consists of a base integer instruction set and several optional extensions. Each extension adds specific functionalities to the processor, such as multiplication, division and floating-point operations.

## A. Pseudo Instructions

Pseudo-instructions are simplified assembly instructions provided by the assembler. They make assembly programs easier to read and write. They are converted into one or more actual RISC-V machine instructions during assembly.

| Instruction | Description |
|---|---|
| `mv` | Moves data from one register to another. |
| `li` | Loads a constant value into a register. |
| `ret` | Returns control to the calling function. |

**Examples:**

- `mv rd, rs` is translated into `addi rd, rs, 0`.
- `li rd, immediate` may expand into one or more instructions depending on the constant value.
- `ret` is an alias for `jalr x0, 0(ra)`.

## B. Base Integer Instructions – RV64I

The RISC-V naming convention is:

- **RV** – RISC-V.
- **64** – 64-bit integer register width (XLEN = 64).
- **I** – Base integer instruction set.

The RV64I base instruction set supports arithmetic, logical, memory-access and control-flow operations.

| Instruction | Description |
|---|---|
| `AUIPC` | Adds an upper immediate value to the program counter and stores the result in a register. |
| `JALR` | Performs an indirect jump and stores the return address. |
| `LUI` | Loads an upper immediate value into a register. |
| `ADDI` | Adds an immediate value to a register. |
| `LW` | Loads a 32-bit word from memory and sign-extends it to 64 bits. |
| `ADDW` | Adds two 32-bit values and sign-extends the result to 64 bits. |
| `SD` | Stores a 64-bit value from a register into memory. |

## C. Multiplication and Division Extension – RV64M

The M extension adds multiplication and division operations to the base integer instruction set.

| Instruction | Description |
|---|---|
| `MULW` | Multiplies two 32-bit values and sign-extends the lower 32-bit result to 64 bits. |
| `DIVW` | Performs signed division on 32-bit values and sign-extends the result to 64 bits. |

**Note:** Combining the base integer instruction set with the multiplication and division extension gives:

`RV64I + RV64M = RV64IM`

## D. Floating-Point Extensions – RV64F and RV64D

RISC-V provides optional floating-point extensions to perform mathematical operations on decimal values.

### 1. RV64F – Single-Precision Floating Point

The F extension supports 32-bit floating-point operations.

| Instruction | Description |
|---|---|
| `FLW` | Loads a 32-bit floating-point value from memory. |
| `FADD.S` | Performs single-precision floating-point addition. |
| `FMUL.S` | Performs single-precision floating-point multiplication. |
| `FDIV.S` | Performs single-precision floating-point division. |
| `FMV.X.W` | Moves the bit representation of a single-precision value into an integer register. |

### 2. RV64D – Double-Precision Floating Point

The D extension supports 64-bit floating-point operations.

| Instruction | Description |
|---|---|
| `FLD` | Loads a 64-bit double-precision floating-point value from memory. |
| `FMV.X.D` | Moves the bit representation of a double-precision value into an integer register on RV64. |

**Note:** `FMUL.S` and `FDIV.S` belong to the F extension, whereas `FLD` and `FMV.X.D` belong to the D extension.

Combining the integer, multiplication/division and floating-point extensions gives:

`RV64I + M + F + D = RV64IMFD`

## E. Application Binary Interface (ABI)

The Application Binary Interface (ABI) defines how compiled programs interact with the processor and other software components.

It specifies register usage, function calling conventions, stack management and data representation.

Some important RISC-V ABI register names are:

| ABI Name | Register | Purpose |
|---|---|---|
| `zero` | x0 | Always contains zero. |
| `ra` | x1 | Stores the return address. |
| `sp` | x2 | Maintains the stack pointer. |
| `gp` | x3 | Global pointer. |
| `tp` | x4 | Thread pointer. |
| `t0–t6` | x5–x7, x28–x31 | Temporary registers. |
| `s0–s11` | x8–x9, x18–x27 | Saved registers. |
| `a0–a7` | x10–x17 | Function arguments; a0 and a1 also hold return values. |

## F. Memory Allocation and Stack Pointer

The stack is a memory region used for storing temporary data, local variables, saved registers and function-call information.

The stack pointer (`sp`) holds the address of the current top of the stack.

In RISC-V, the stack grows towards lower memory addresses.

## 3. Experiment 1 – C Program to Calculate Sum of Numbers from 1 to 5

### Objective

To write and execute a C program that calculates the sum of natural numbers from 1 to 5 using a `for` loop and understand the basic compilation and execution process.

### Source Code

```c
#include <stdio.h>

int main()
{
    int i, sum = 0, n = 5;

    for (i = 1; i <= n; ++i)
    {
        sum += i;
    }

    printf("Sum of numbers from 1 to %d is %d\n", n, sum);

    return 0;
}
```

### Program Explanation

- `#include <stdio.h>` – Includes the standard input/output library.
- `int sum = 0` – Initializes the sum variable to zero.
- `n = 5` – Sets the upper limit to 5.
- `for (i = 1; i <= n; ++i)` – Iterates from 1 to 5.
- `sum += i` – Adds the current value of `i` to the sum.
- `printf()` – Displays the final result.

### Compilation

The program is compiled using the GCC compiler.

```bash
gcc sum1ton.c
```

### Execution

Run the generated executable using:

```bash
./a.out
```

### Output

```text
Sum of numbers from 1 to 5 is 15
```

### Screenshots

**1. C Source Code and output**
<img width="955" height="910" alt="output" src="https://github.com/user-attachments/assets/288b2cd4-5e0f-4dd8-9cfe-74205c502abd" />



### Result

The C program executes successfully and calculates the sum of natural numbers from 1 to 5. The output is **15**.
### 4. Compilation Using RISC-V GCC – O1 Optimization

After successfully compiling and executing the C program using the native GCC compiler, the same program is compiled using the **RISC-V GNU Cross-Compiler**.

A cross-compiler generates executable code for a target architecture different from the host system. In this experiment, the host system uses Ubuntu, while the target architecture is RISC-V 64-bit (RV64I).

**Compilation Command:**

```bash
riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c
```

### Command Explanation

| Option | Description |
|---|---|
| `riscv64-unknown-elf-gcc` | RISC-V cross-compiler used to generate code for a RISC-V target. |
| `-O1` | Enables level 1 compiler optimization to improve code efficiency. |
| `-mabi=lp64` | Specifies the LP64 Application Binary Interface, with 64-bit long integers and pointers. |
| `-march=rv64i` | Targets the RV64I base integer instruction set. |
| `-o sum1ton.o` | Specifies the output filename. |
| `sum1ton.c` | Input C source file. |

### Understanding O1 Optimization

The `-O1` option enables basic compiler optimizations. It helps reduce unnecessary instructions and improves program execution efficiency without applying the more aggressive optimizations used by higher optimization levels.

### Important Note

Although the output filename uses the `.o` extension, the command does not include the `-c` option. Therefore, the compiler driver performs the linking stage and generates an executable rather than a relocatable object file.

The filename extension alone does not determine the actual file type.

### Screenshot – RISC-V O1 Compilation
<img width="1920" height="1080" alt="Screenshot 2026-11-23 154222" src="https://github.com/user-attachments/assets/954acb1f-f32d-4073-ba4c-1605f43b43a1" />


### Result

The C program is successfully cross-compiled using the RISC-V GNU Compiler with O1 optimization, targeting the RV64I architecture.
### 5. Compilation Using RISC-V GCC – Ofast Optimization

After compiling the program using the `-O1` optimization level, the same C program is compiled again using the `-Ofast` optimization option.

The `-Ofast` option enables aggressive compiler optimizations to improve program performance. It includes optimizations from `-O3` along with additional options that may relax strict language-standard and floating-point rules.

**Compilation Command:**

```bash
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c
```

### Command Explanation

| Option | Description |
|---|---|
| `riscv64-unknown-elf-gcc` | RISC-V cross-compiler used to generate code for the target architecture. |
| `-Ofast` | Enables aggressive optimization options beyond the standard `-O3` level. |
| `-mabi=lp64` | Specifies the LP64 Application Binary Interface. |
| `-march=rv64i` | Targets the RV64I base integer instruction set. |
| `-o sum1ton.o` | Specifies the output filename. |
| `sum1ton.c` | Input C source file. |

### Understanding Ofast Optimization

- Enables aggressive compiler optimizations to improve execution performance.
- Includes optimizations available at the `-O3` level.
- May relax strict floating-point and language-standard compliance rules.
- Attempts to reduce unnecessary operations and generate efficient machine instructions.

### Instruction-Level Analysis

After compilation, the generated RISC-V instructions can be examined using the Objdump utility.

```bash
riscv64-unknown-elf-objdump -d sum1ton.o
```

This helps inspect the assembly instructions generated by the compiler and understand how optimization affects the machine code.
### Screenshot – RISC-V Ofast Compilation
<img width="1920" height="990" alt="Screenshot 2026-11-23 162023" src="https://github.com/user-attachments/assets/93614913-9ffc-477d-96be-189fdd9c3513" />

### Result

The C program is successfully compiled using the RISC-V GNU Compiler with `-Ofast` optimization, targeting the RV64I architecture. The generated instructions are inspected to understand the effect of aggressive compiler optimization.
### 6. Disassembling the RISC-V Executable

After compiling the C program using the RISC-V GCC compiler, the `objdump` utility is used to inspect the generated machine code in a readable assembly format.

This process helps understand how high-level C statements are translated into RISC-V assembly instructions.

**Command:**

```bash
riscv64-unknown-elf-objdump -d sum1ton.o | less
```

### Command Explanation

| Command | Description |
|---|---|
| `riscv64-unknown-elf-objdump` | RISC-V binary inspection utility. |
| `-d` | Disassembles executable sections into assembly instructions. |
| `sum1ton.o` | Compiled RISC-V executable. |
| `\|` | Passes the output of one command to another. |
| `less` | Displays output page by page for easy inspection. |

### Understanding the Disassembly Output

The disassembly output contains:

- **Address:** Memory address of each instruction.
- **Machine Code:** Encoded hexadecimal representation of the instruction.
- **Assembly Instruction:** Human-readable RISC-V instruction.
- **Operands:** Registers or immediate values used by the instruction.

This helps inspect the actual instructions generated by the compiler.

**To exit the `less` viewer:**

Press:

```text
q
```

---

### 7. Simulating the RISC-V Program Using Spike

After generating and inspecting the RISC-V executable, the program is executed using the **Spike RISC-V Instruction Set Simulator**.

Spike simulates the behavior of a RISC-V processor and executes the compiled instructions without requiring physical RISC-V hardware.

**Execution Command:**

```bash
spike pk sum1ton.o
```

### Command Explanation

| Component | Description |
|---|---|
| `spike` | RISC-V instruction set simulator. |
| `pk` | Proxy kernel that provides the execution environment for the program. |
| `sum1ton.o` | Compiled RISC-V executable. |

### Working Process

1. Spike starts the RISC-V instruction set simulator.
2. The proxy kernel (`pk`) loads and runs the program.
3. The processor instructions execute inside the simulator.
4. The program displays the calculated sum.

### Expected Output

```text
bbl loader
Sum of numbers from 1 to 100 is 5050
```

The output is compared with the result obtained from native GCC execution to verify that the RISC-V compiled program produces the expected result.

### Screenshot – Spike Simulation

<img width="1920" height="923" alt="VirtualBox_riscv_teju_23_11_2026_16_25_57" src="https://github.com/user-attachments/assets/074a1f13-684a-4a59-8cc2-10d3cfcc120b" />


### Result

The RISC-V executable is successfully disassembled using Objdump and executed using the Spike simulator. The simulation produces the expected output of **5050**, confirming the program's correct execution.
## 8. Experiment – Understanding Signed and Unsigned Integer Representation

### Objective

To understand how signed and unsigned integer data types represent values in C programming and explore the minimum and maximum values that can be stored in a 64-bit integer.

### 8.1 Signed Integer Representation

Signed integers represent both positive and negative numbers, including zero.

RISC-V uses the two's complement representation for signed integer values.

For an integer with N bits, the range is:

- **Minimum value:** `-2^(N-1)`
- **Maximum value:** `2^(N-1) - 1`

For a 64-bit signed integer:

- Minimum value: `-9223372036854775808`
- Maximum value: `9223372036854775807`

**Source File:** `signedHighest.c`

### Explanation

- A signed 64-bit integer uses one bit as the sign position.
- It represents both positive and negative values.
- Two's complement representation is used to represent negative numbers.
- The maximum positive value is one less than `2^63`.

### Screenshot – Signed Integer Representation

<img width="1920" height="923" alt="signed code" src="https://github.com/user-attachments/assets/d1a1a8b6-a527-446b-831d-3d4944b71e45" />


### 8.2 Unsigned Integer Representation

Unsigned integers represent only non-negative values, including zero.

For an unsigned integer with N bits, the range is:

- **Minimum value:** `0`
- **Maximum value:** `2^N - 1`

For a 64-bit unsigned integer:

- Minimum value: `0`
- Maximum value: `18446744073709551615`

**Source File:** `unsignedHighest.c`

### Explanation

- All 64 bits are used to represent the numerical value.
- There is no sign bit.
- It represents only zero and positive values.
- The maximum value is `2^64 - 1`.

### Screenshot – Unsigned Integer Representation
<img width="1920" height="923" alt="VirtualBox_riscv_teju_23_11_2026_21_43_59" src="https://github.com/user-attachments/assets/87f702d6-1ea3-4536-8827-b8accc5dc20f" />


---
<img width="1920" height="923" alt="unsignedoutput" src="https://github.com/user-attachments/assets/ac8e18d4-3a18-472c-823a-c8a75ddbd450" />

### 8.3 Comparison Between Signed and Unsigned Integers

| Feature | Signed Integer | Unsigned Integer |
|---|---|---|
| Represents | Positive and negative values | Only non-negative values |
| Minimum value (64-bit) | -9223372036854775808 | 0 |
| Maximum value (64-bit) | 9223372036854775807 | 18446744073709551615 |
| Representation | Two's complement | Binary representation |
| Sign bit | MSB indicates the sign | No separate sign bit |

### Result

The experiment demonstrates the difference between signed and unsigned integer representations in C. It helps understand how a 64-bit data type stores numerical values and how the available bit positions determine the minimum and maximum representable values.
# Learning Outcomes

Through Day 1 of the RISC-V workshop, I gained knowledge of the following concepts:

1. **RISC-V ISA Fundamentals:** Understood how the Instruction Set Architecture (ISA) acts as an interface between software and hardware.

2. **Instruction Classification:** Explored different types of RISC-V instructions, including pseudo instructions, RV64I, RV64M, and floating-point extensions such as RV64F and RV64D.

3. **ABI and Registers:** Understood the purpose of registers such as `a0`, `sp`, `ra`, and `s0`, and their roles in function calls and program execution.

4. **GNU Compiler Toolchain:** Learned how to compile C programs for a RISC-V target using the RISC-V cross-compiler.

5. **Compiler Optimization:** Explored the `-O1` and `-Ofast` optimization levels and understood how compiler optimization affects the generated machine instructions.

6. **Disassembly and Simulation:** Used `objdump` to inspect generated RISC-V assembly instructions and the Spike simulator to execute RISC-V programs.

## Conclusion

Day 1 provided a practical understanding of how a high-level C program is converted into RISC-V machine instructions and executed by a processor.

By exploring the ISA, ABI, GNU compiler toolchain, compiler optimization levels, disassembly, and Spike simulator, I gained a clear understanding of the software-to-hardware execution flow. These concepts provide a strong foundation for learning RISC-V processor architecture and advanced topics in the upcoming sessions.
