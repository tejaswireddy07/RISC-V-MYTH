# Day 2 – Application Binary Interface (ABI)

## 1. Introduction

RISC-V is an open-standard Instruction Set Architecture (ISA) based on Reduced Instruction Set Computer (RISC) principles. It provides a simple, modular and extensible architecture for processor design.

Day 1 focuses on understanding the software-to-hardware execution flow using C programming, RISC-V assembly language, the GNU compiler toolchain, object file generation, assembly-level code inspection and Spike simulation.

Day 2 focuses on the **Application Binary Interface (ABI)** and its role in RISC-V program execution. ABI defines the rules that allow software components to work together. It specifies how function arguments and return values are passed, how registers are used, how the stack is managed and how functions communicate with each other.

The practical session explores RISC-V function calls, register usage, memory operations and basic verification flow. It provides a better understanding of how C programs interact with RISC-V assembly code and how processor instructions support program execution.
## 2. Understanding the RISC-V GNU Compiler Toolchain

The **GNU Compiler Toolchain** provides the tools required to compile, assemble, link, execute and inspect programs developed for RISC-V processors. It converts C and assembly source code into executable machine code that a RISC-V processor can understand.

### 2.1 Important Tools

| **Tool** | **Purpose** |
|---|---|
| RISC-V GCC | Compiles C programs for the RISC-V architecture. |
| GNU Assembler | Converts assembly language code into object files. |
| GNU Linker | Combines object files and required libraries to generate an executable file. |
| GNU Objdump | Displays disassembled machine instructions from executable or object files. |
| Spike | Simulates the execution of RISC-V instructions. |
| Proxy Kernel (`pk`) | Provides a basic execution environment for RISC-V application programs running on Spike. |

### 2.2 Compilation Flow

The basic RISC-V compilation and execution flow involves the following steps:

1. **Write the C Program:** Create the main program using C language.
2. **Write the Assembly Program:** Create the required RISC-V assembly function.
3. **Compile the Source Files:** Use the RISC-V GCC cross-compiler to compile the C and assembly files.
4. **Generate the Executable:** Link the compiled files to generate a RISC-V ELF executable.
5. **Execute Using Spike:** Run the executable using the Spike simulator and Proxy Kernel.
6. **Inspect Machine Instructions:** Use GNU Objdump to examine the generated RISC-V assembly instructions.
## 3. Practical Implementation – Sum of Numbers Using C and RISC-V Assembly

### 3.1 Objective

To implement a program that calculates the sum of numbers using a C program and an external RISC-V assembly function. This experiment demonstrates how a C program calls an assembly function and receives the calculated result through the Application Binary Interface (ABI).

### 3.2 C Program – `1to9_custom.c`

The C program calls an external assembly function named `load()` to calculate the sum.

**Source Code:**

```c
#include <stdio.h>

extern int load(int x, int y);

int main()
{
    int result = 0;
    int count = 9;

    result = load(0, count + 1);

    printf("Sum of numbers from 1 to %d is %d\n",
           count, result);

    return 0;
}
```

**Program Explanation:**

- `stdio.h` – Includes the standard input/output library for the `printf()` function.
- `extern int load(int x, int y);` – Declares an external function implemented in RISC-V assembly.
- `result` – Stores the value returned by the assembly function.
- `count = 9` – Sets the upper limit of the calculation.
- `load(0, count + 1)` – Calls the assembly function with arguments 0 and 10.
- `printf()` – Displays the calculated result.

**Screenshot – C Program**

<img width="955" height="910" alt="day2 ss1" src="https://github.com/user-attachments/assets/7f78bc1e-9091-4900-8c73-2555b7c787cd" />


### 3.3 RISC-V Assembly Program – `load.S`

The assembly program implements the addition operation using RISC-V instructions. It receives arguments through registers, performs the addition inside a loop and returns the final result to the C program.

**Assembly Code:**

```asm
.section .text
.global load
.type load, @function

load:
    add  a4, a0, zero
    add  a2, a0, a1
    add  a3, a0, zero

loop:
    add  a4, a3, a4
    addi a3, a3, 1
    blt  a3, a2, loop

    add  a0, a4, zero
    ret
```

**Instruction Explanation:**

| Instruction | Purpose |
|---|---|
| `add` | Adds two register values. |
| `addi` | Adds an immediate value to a register. |
| `blt` | Branches to a label if one register value is less than another. |
| `ret` | Returns control to the calling function. |
| `zero` | Represents the constant-zero register (`x0`). |
| `a0`, `a1` | Hold function arguments; `a0` also holds the return value. |
| `a2`, `a3`, `a4` | Store the loop limit, current number and accumulated sum. |

**Working Principle:**

1. The assembly function receives the arguments through `a0` and `a1`.
2. It initializes the sum and loop counter.
3. The loop adds the current number to the accumulated sum.
4. The `addi` instruction increments the counter.
5. The `blt` instruction repeats the loop while the counter is less than the limit.
6. The final sum is placed in `a0`.
7. The `ret` instruction returns the result to the C program.

**Screenshot – Assembly Program**

<img width="955" height="910" alt="day2 ss2" src="https://github.com/user-attachments/assets/a9334155-2af2-463a-8694-fb516a84eca7" />

### 3.4 Expected Output

```text
Sum of numbers from 1 to 9 is 45
```

**Note:** The assembly function adds the values from 0 to 9. Since adding zero does not change the sum, the result is 45.

### 3.5 Result

The C program successfully calls the external RISC-V assembly function using the ABI calling convention. The assembly function performs the addition and returns the result to the C program through register `a0`.
## 4. Compilation Using RISC-V GCC

The RISC-V GCC cross-compiler converts the C and assembly source files into a RISC-V executable ELF file. This executable can run on the Spike instruction set simulator.

### 4.1 Compilation Command

**Command:**

```bash
riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o 1to9_custom.elf 1to9_custom.c load.S
```

**Command Explanation:**

| Option | Purpose |
|---|---|
| `riscv64-unknown-elf-gcc` | RISC-V GCC cross-compiler. |
| `-O1` | Enables basic compiler optimizations. |
| `-mabi=lp64` | Specifies the 64-bit integer and pointer ABI. |
| `-march=rv64i` | Targets the RV64I base instruction set. |
| `-o 1to9_custom.elf` | Specifies the output executable filename. |
| `1to9_custom.c` | C source file. |
| `load.S` | RISC-V assembly source file. |

### 4.2 Execution Using Spike Simulator

After successful compilation, execute the generated ELF file using the Spike simulator.

**Command:**

```bash
spike pk 1to9_custom.elf
```

**Command Explanation:**

- `spike` – Starts the RISC-V instruction set simulator.
- `pk` – Loads the Proxy Kernel to provide an execution environment.
- `1to9_custom.elf` – Specifies the generated RISC-V executable.

### 4.3 Expected Output

```text
Sum of numbers from 1 to 9 is 45
```

The output displays the sum calculated by the RISC-V assembly function and returned to the C program.

### 4.4 Screenshot – Compilation and Execution


<img width="955" height="910" alt="day2 ss3" src="https://github.com/user-attachments/assets/0a6a531d-3c14-4994-b48f-4f11cd156313" />

### 4.5 Result

The C and RISC-V assembly programs are compiled successfully using the RISC-V GCC cross-compiler. The generated ELF executable runs on the Spike simulator and displays the expected sum of 45.
## 5. Understanding Object Files and Disassembly

After compilation, the generated ELF executable contains RISC-V machine instructions. These instructions can be inspected using **GNU Objdump**, which converts machine code into a human-readable assembly representation.

### 5.1 Viewing Disassembled Instructions

**Command:**

```bash
riscv64-unknown-elf-objdump -d 1to9_custom.elf | less
```

**Command Explanation:**

| Command | Purpose |
|---|---|
| `riscv64-unknown-elf-objdump` | Runs the RISC-V disassembly tool. |
| `-d` | Disassembles executable sections. |
| `1to9_custom.elf` | Specifies the compiled ELF executable. |
| `\|` | Sends the command output to another program. |
| `less` | Displays the disassembly output page by page. |

**Note:** Press `q` to exit the `less` viewer and return to the terminal.

### 5.2 Understanding Disassembly

Disassembly helps us understand how high-level C instructions are translated into low-level RISC-V machine instructions.

It provides information about:

- **Instruction Addresses:** Memory locations of instructions.
- **Machine Code:** Binary instruction representations displayed in hexadecimal format.
- **Assembly Instructions:** Human-readable RISC-V instructions.
- **Register Usage:** Registers involved in calculations and data movement.
- **Function Calls and Returns:** Instructions used to call and return from functions.
- **Branch and Arithmetic Operations:** Instructions used for loops, conditions and mathematical calculations.

### 5.3 Importance of Disassembly

- Helps understand compiler-generated assembly code.
- Shows how C functions interact with assembly functions.
- Helps identify register usage and instruction sequences.
- Provides insight into program execution at the processor level.
- Supports debugging and performance analysis.

### 5.4 Screenshot – RISC-V Disassembly
<img width="955" height="910" alt="day2 ss4" src="https://github.com/user-attachments/assets/b2169b37-913a-4243-a2d6-4c156b2588f7" />


### 5.5 Result

The generated RISC-V ELF executable is inspected using GNU Objdump. The disassembly output provides a readable view of the machine instructions, register usage and program control flow.
## 6. Exploring RISC-V Workshop Collateral

The RISC-V workshop collateral directory contains the source files, execution scripts, testbench files and toolchain resources required for the practical session.

### 6.1 Exploring the Directory

Navigate to the workshop collateral directory:

```bash
cd ~/riscv_workshop_collaterals
ls -ltr
```

- `cd` – Opens the specified directory.
- `ls -ltr` – Lists files and directories in time order.

### 6.2 Exploring the Labs Directory

Navigate to the labs directory:

```bash
cd labs
ls -ltr
```

The labs directory contains files such as:

| File | Description |
|---|---|
| `testbench.v` | Verilog testbench |
| `syscalls.c` | System call implementation |
| `start.S` | Startup assembly code |
| `rv32im.sh` | RISC-V execution script |
| `riscv.ld` | Linker script |
| `picorv32.v` | PicoRV32 processor source |
| `hello.c` | C program |
| `1to9_custom.c` | Custom C program |
| `load.S` | Assembly function |

### 6.3 Screenshot – Workshop Directory Structure
<img width="955" height="910" alt="day2 ss5" src="https://github.com/user-attachments/assets/d6140d49-834f-4955-ba21-ec328e9f0c7d" />


## 7. Understanding the RISC-V Execution Script

The `rv32im.sh` script contains the commands required to compile the source files, link the object files and generate the firmware image for processor simulation.

### 7.1 Compilation and Linking Flow

The script performs the following operations:

1. Compiles the C source file.
2. Compiles the RISC-V assembly source file.
3. Compiles the system call implementation.
4. Links the object files using the linker script.
5. Generates the ELF executable.
6. Converts the ELF file into a Verilog-compatible firmware image.
7. Compiles the Verilog testbench and processor source.

### 7.2 Execution Script – `rv32im.sh`

The script uses commands similar to the following:

```bash
riscv64-unknown-elf-gcc -c -mabi=ilp32 -march=rv32im -o 1to9_custom.o 1to9_custom.c

riscv64-unknown-elf-gcc -c -mabi=ilp32 -march=rv32im -o load.o load.S

riscv64-unknown-elf-gcc -c -mabi=ilp32 -march=rv32im -o syscalls.o syscalls.c

riscv64-unknown-elf-gcc -mabi=ilp32 -march=rv32im -Wl,--gc-sections -o firmware.elf 1to9_custom.o load.o syscalls.o -T riscv.ld -lstdc++

riscv64-unknown-elf-objcopy -O verilog firmware.elf firmware.tmp

python3 hex8tohex32.py firmware.hex > firmware32.hex

iverilog -o testbench.vvp testbench.v picorv32.v

vvp -N testbench.vvp
```

### 7.3 Explanation of Important Commands

| Command | Purpose |
|---|---|
| `-c` | Compiles the source file into an object file without linking. |
| `-mabi=ilp32` | Specifies the 32-bit integer, long and pointer ABI. |
| `-march=rv32im` | Targets the RV32IM instruction set. |
| `-Wl,--gc-sections` | Passes the option to the linker to remove unused sections. |
| `-T riscv.ld` | Specifies the linker script. |
| `objcopy` | Converts the ELF file into another output format. |
| `iverilog` | Compiles the Verilog testbench and processor source. |
| `vvp` | Executes the compiled Verilog simulation. |

### 7.4 Screenshot – RISC-V Execution Script
<img width="955" height="910" alt="day2 ss6" src="https://github.com/user-attachments/assets/1c706957-285c-4277-a255-26ad195289a8" />

<img width="955" height="910" alt="day2 ss7" src="https://github.com/user-attachments/assets/2ba0ba6f-4b7c-4cd7-be0e-3cf5a065bb9e" />

## 8. Modifying and Running the C Program

In this experiment, the C program is modified to change the number of terms used in the summation.

### 8.1 Modified C Program

The `count` variable is changed from 9 to 3.

**Source Code:**

```c
#include <stdio.h>

extern int load(int x, int y);

int main()
{
    int result = 0;
    int count = 3;

    result = load(0, count + 1);

    printf("Sum of number from 1 to %d is %d\n",
           count, result);

    return 0;
}
```

**Program Explanation:**

- `count = 3` sets the upper limit to 3.
- `load(0, count + 1)` calls the external assembly function with arguments 0 and 4.
- The assembly function calculates the sum from 0 to 3.
- The returned result is displayed using `printf()`.

### 8.2 Execution

First, provide execution permission to the script:

```bash
chmod +x rv32im.sh
```

Run the script:

```bash
./rv32im.sh
```

### 8.3 Output

```text
Sum of number from 1 to 3 is 6
TRAP
```

The calculated sum is:

**1 + 2 + 3 = 6**

### 8.4 Screenshot – Modified Program Execution
<img width="955" height="910" alt="day2 ss8" src="https://github.com/user-attachments/assets/55c4a3f4-05e1-4195-a0b8-4dd53a585e43" />

<img width="1208" height="1302" alt="day2 ss9" src="https://github.com/user-attachments/assets/cb236abf-2b7c-45bb-a31d-bdb73bd3ed9f" />


## 9. Firmware Generation and Inspection

Firmware generation converts the compiled program into a format that the processor simulation environment can load and execute.

### 9.1 Firmware Files

The following files are generated during the build process:

| File | Description |
|---|---|
| `firmware.elf` | Compiled and linked executable file. |
| `firmware.hex` | Hexadecimal firmware representation. |
| `firmware32.hex` | Converted 32-bit hexadecimal firmware data. |
| `testbench.vvp` | Compiled Verilog simulation executable. |

### 9.2 Inspecting Firmware Hexadecimal Data

Use the following command to open the firmware hexadecimal file:

```bash
vim firmware.hex
```

The hexadecimal file contains the machine-code representation of the compiled program in a format used by the simulation environment.

### 9.3 Screenshot – Firmware Hexadecimal Output
<img width="955" height="910" alt="day2 ss10" src="https://github.com/user-attachments/assets/31e0fb0e-4bce-49fd-acb7-2860396976d1" />

<img width="955" height="910" alt="day2 ss11" src="https://github.com/user-attachments/assets/f7b1ccfe-edb7-46fd-92a9-144ab67054de" />

## 10. Verilog Testbench and Processor Simulation

A Verilog testbench provides a simulation environment for testing the PicoRV32 processor.

### 10.1 Testbench – `testbench.v`

The testbench instantiates the PicoRV32 processor and connects the required signals, such as:

- Clock signal
- Reset signal
- Memory interface signals

It provides the environment required to simulate processor operation.

### 10.2 Running the Simulation

First, compile the Verilog testbench and PicoRV32 processor source:

```bash
iverilog -o testbench.vvp testbench.v picorv32.v
```

Next, execute the generated simulation file:

```bash
vvp -N testbench.vvp
```

**Command Explanation:**

| Command | Purpose |
|---|---|
| `iverilog` | Verilog compiler. |
| `-o testbench.vvp` | Specifies the output simulation filename. |
| `testbench.v` | Verilog testbench source. |
| `picorv32.v` | PicoRV32 processor source. |
| `vvp` | Runs the compiled simulation. |
| `-N` | Uses non-interactive simulation mode. |

### 10.3 Screenshot – Verilog Testbench

<img width="955" height="910" alt="day2 ss12" src="https://github.com/user-attachments/assets/2c6a80eb-9f15-4553-b415-1d7deb58355b" />

## 11. Tools and Technologies Used

The following tools and technologies are used during the practical session:

| Tool / Technology | Purpose |
|---|---|
| Ubuntu Linux | Development environment |
| RISC-V GCC | Cross-compilation |
| GNU Assembler | Assembly compilation |
| GNU Objdump | Disassembly analysis |
| Spike Simulator | RISC-V instruction simulation |
| Proxy Kernel | Application execution environment |
| Icarus Verilog | Verilog compilation |
| VVP | Simulation execution |
| PicoRV32 | RISC-V processor core |
| C Programming | Application development |
| RISC-V Assembly | Low-level programming |

---

## 12. Key Learning Outcomes

Through Day 2 of the RISC-V workshop, I gained knowledge of the following concepts:

1. **Application Binary Interface:** Understood how C programs communicate with RISC-V assembly functions.
2. **Function Calls:** Learned how function arguments and return values are transferred through registers.
3. **GNU Compiler Toolchain:** Practiced compiling and linking C and assembly source files.
4. **RISC-V Execution Script:** Explored the commands used for compilation, linking and firmware generation.
5. **Program Modification:** Modified the C program to calculate the sum of different numbers.
6. **Firmware Generation:** Understood how executable files are converted into hexadecimal firmware data.
7. **Verilog Testbench:** Explored how a testbench provides a simulation environment for a processor.
8. **Icarus Verilog:** Practiced compiling and executing Verilog simulation files.
9. **PicoRV32:** Explored the RISC-V processor source and its role in hardware simulation.

---

## 13. Conclusion

Day 2 provides practical experience with the Application Binary Interface (ABI), C and RISC-V assembly interaction, and the GNU compiler toolchain.

The experiments demonstrate how C programs call assembly functions, how function arguments and return values are transferred through registers, and how compiler scripts generate executable and firmware files.

The session also introduces firmware inspection and processor simulation using Verilog, Icarus Verilog and the PicoRV32 processor core.

These practical exercises strengthen the understanding of RISC-V software execution, firmware generation and basic processor verification.
