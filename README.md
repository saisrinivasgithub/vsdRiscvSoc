# vsdRiscvSoc
## How to Install and Unpack the RISC-V Toolchain
Installation guide for the RISC-V GNU toolchain targeting RV64 bare-metal systems. This version fixes common build errors and includes essential troubleshooting steps.
# 1 Prerequisites

```bash
#Ubuntu
sudo apt update
sudo apt install autoconf automake autotools-dev curl python3 
```
# 2 Corrected Installation Steps
## 2.1 Create Workspace
```bash
export WORKSPACE=$HOME/riscv-toolchain
mkdir -p $WORKSPACE
cd $WORKSPACE
```
## 2.2 Clone Repository with Fixes
```bash
git clone --depth 1 --branch 2024.04.05\
https :// github . com / riscv -collab / riscv -gnu -toolchain . git
cd riscv -gnu -toolchain
git submodule update --init --recursive --depth 1 --jobs 8
```
## 2.3 Configure with Fixed Parameters
```bash
./configure --prefix=/opt/riscv64 - elf \
--with -arch=rv64imac \
--with -abi=lp64 \
--with -cmodel=medany \
--with -newlib \
--without -headers \
--disable -shared \
--disable -threads \
--disable -gdb \
--disable -libssp \
--disable -libquadmath
```
##  2.4 Build with Fixed Parallelism
```bash
PARALLEL_JOBS =$(($(nproc)/2))
make -j$PARALLEL_JOBS
sudo make install
```
## 2.5 Environment Setup with Verification
```bash
echo ’ export PATH=/opt/riscv64 -elf/ bin : $PATH ’ >> ~/.bashrc
source ~/.bashrc
riscv64 -unknown -elf -gcc --version
```
# 3 Critical Troubleshooting Section
# Fix common build failures:
## 3.1 Submodule Fixes
```bash
cd riscv -gnu -toolchain
git submodule deinit -f .
git submodule update --init --recursive --depth 1 --jobs 8
```
## 3.2 Memory Error Fix
```bash
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```
## 3.3 Compiler Error Fix
```bash
export CXXFLAGS ="-O1"
make clean
make -j$PARALLEL_JOBS
```
## 3.4 Missing Header Fix
```bash
sudo ln -s / usr / include / x86_64 -linux -gnu / bits / usr / include / bits
sudo ln -s / usr / include / x86_64 -linux -gnu / gnu / usr / include / gnu
sudo ln -s / usr / include / x86_64 -linux -gnu / sys / usr / include / sys
```
# 4 Test Toolchain with Fixed Example
```bash
cat > test .c << EOF
int _start () {
return 0;
}
EOF
riscv64 -unknown -elf -gcc -march = rv64imac -mabi = lp64 \
-mcmodel = medany -nostartfiles - ffreestanding \
-Wl ,-Ttext =0 x80000000 -o test . elf test . c
file test . elf
```
![Screenshot](Screenshot%20from%202025-06-06%2016-28-53.png)
![Screenshot](Screenshot%20from%202025-06-08%2013-10-33.png)
# Compile “Hello, RISC-V” 
“Show me a minimal C ‘hello world’ that cross-compiles for RV32IMC and the exact gcc
flags to produce an ELF.”
# AI Response
hello.c code
```c
/ Minimal bare-metal RISC-V UART output
#define UART_BASE 0x10000000  // Standard QEMU virt machine UART address

void uart_putc(char c) {
    volatile char *uart = (volatile char *)UART_BASE;
    *uart = c;
}


void uart_puts(const char *s) {
    while (*s) {
        uart_putc(*s++);
    }
}

int main() {
    uart_puts("Hello, RISC-V World!\n");
    return 0;
}

void _start() {
    main();
    while (1);  // Halt CPU
}
```
link.ld code
```c
NTRY(_start)

MEMORY {
    RAM (rwx) : ORIGIN = 0x80000000, LENGTH = 128K
}

SECTIONS {
    .text : {
        *(.text._start)
        *(.text*)
    } > RAM

    .data : { *(.data*) } > RAM
    .bss : { *(.bss*) } > RAM
    _end = .;
}
```
Build Commands
```bash
riscv64-unknown-elf-gcc -march=rv32imac -mabi=ilp32 \
    -nostdlib -ffreestanding -Tlink.ld \
    -o hello.elf hello.c

# Convert to raw binary (optional)
riscv64-unknown-elf-objcopy -O binary hello.elf hello.bin
```
### Key Details:
### UART Address:
0x10000000 is standard for QEMU's virt machine
Change if targeting different hardware
### Memory Map:
Text section starts at 0x80000000 (common RISC-V boot address)
128KB RAM is sufficient for this example
### Bare-Metal Essentials:
No standard library (-nostdlib)
No CRT startup (-ffreestanding)
Custom _start that calls main()
### To Run in QEMU:
```bash
qemu-system-riscv32 -nographic -machine virt \
    -bios none -kernel hello.elf
```
### Expected Output:
![Screenshot](./Screenshot%20from%202025-06-08%2013-01-25.png)
# From C to Assembly
## How do I generate the .s file and explain the prologue/epilogue of the main function?
### 1. Generate Assembly File (.s)
Use GCC with -S flag to produce assembly output:
```bash
riscv64-unknown-elf-gcc -S -march=rv32imac -mabi=ilp32 -O0 hello.c -o hello.s
```
- -S: Stop after generating assembly (instead of object/ELF files)
- -O0: Disable optimizations (for clearer prologue/epilogue)
### 2. Assembly Output (Main Function)
###  uart_putc Function (Address: 0x80000000)
```asm
80000000:	7179                	addi	sp,sp,-48      # Allocate 48 bytes stack space
80000002:	d622                	sw	s0,44(sp)      # Save frame pointer (s0) on stack
80000004:	1800                	addi	s0,sp,48      # Set new frame pointer (s0 = sp + 48)
80000006:	87aa                	mv	a5,a0         # Move char argument (a0) to a5
80000008:	fcf40fa3          	sb	a5,-33(s0)    # Store char on stack (byte)
8000000c:	100007b7          	lui	a5,0x10000    # Load UART base addr (0x10000 << 12)
80000010:	fef42623          	sw	a5,-20(s0)    # Store UART addr on stack
80000014:	fec42783          	lw	a5,-20(s0)    # Reload UART addr
80000018:	fdf44703          	lbu	a4,-33(s0)   # Load char from stack
8000001c:	00e78023          	sb	a4,0(a5)      # Write char to UART (*(a5) = a4)
80000020:	0001                	nop             # No-operation (padding)
80000022:	5432                	lw	s0,44(sp)    # Restore frame pointer
80000024:	6145                	addi	sp,sp,48    # Deallocate stack
80000026:	8082                	ret             # Return
```
### Key Points:
- Stack frame is oversized (48 bytes) due to -O0 (no optimizations)
- UART address (0x10000000) is loaded in two steps:
1. lui loads upper 20 bits
2. sb stores the byte to the UART
### uart_puts Function (Address: 0x80000028)
```asm
80000028:	1101                	addi	sp,sp,-32    # Allocate 32 bytes stack
8000002a:	ce06                	sw	ra,28(sp)     # Save return address
8000002c:	cc22                	sw	s0,24(sp)     # Save frame pointer
8000002e:	1000                	addi	s0,sp,32     # Set new frame pointer
80000030:	fea42623          	sw	a0,-20(s0)    # Store string pointer on stack
80000034:	a819                	j	8000004a      # Jump to loop condition (first time)
80000036:	fec42783          	lw	a5,-20(s0)    # Load string pointer
8000003a:	00178713          	addi	a4,a5,1      # Increment pointer (a7 = a5 + 1)
8000003e:	fee42623          	sw	a4,-20(s0)    # Store updated pointer
80000042:	0007c783          	lbu	a5,0(a5)     # Load current char (a5 = *a5)
80000046:	853e                	mv	a0,a5        # Move char to a0 (argument)
80000048:	3f65                	jal	80000000     # Call uart_putc (PC-relative)
8000004a:	fec42783          	lw	a5,-20(s0)    # Load string pointer
8000004e:	0007c783          	lbu	a5,0(a5)     # Load current char
80000052:	f3f5                	bnez	a5,80000036  # Loop if char != '\0'
80000054:	0001                	nop             # Padding
80000056:	0001                	nop             # Padding
80000058:	40f2                	lw	ra,28(sp)    # Restore return address
8000005a:	4462                	lw	s0,24(sp)    # Restore frame pointer
8000005c:	6105                	addi	sp,sp,32    # Deallocate stack
8000005e:	8082                	ret             # Return
```
### Loop Structure:
1. Initialization: Jump directly to condition check (0x8000004a)
2. Body: Load char → print → increment pointer (0x80000036-0x80000048)
3. Condition: Check for null terminator (0x8000004a-0x80000052)
### main Function (Address: 0x80000060)
```asm
80000060:	1141                	addi	sp,sp,-16    # Allocate 16 bytes stack
80000062:	c606                	sw	ra,12(sp)     # Save return address
80000064:	c422                	sw	s0,8(sp)      # Save frame pointer
80000066:	0800                	addi	s0,sp,16     # Set frame pointer
80000068:	800007b7          	lui	a5,0x80000    # Load upper bits of string addr
8000006c:	08c78513          	addi	a0,a5,140    # a0 = 0x8000008c (string addr)
80000070:	3f65                	jal	80000028     # Call uart_puts
80000072:	4781                	li	a5,0         # Load return value (0)
80000074:	853e                	mv	a0,a5        # Move to return register (a0)
80000076:	40b2                	lw	ra,12(sp)    # Restore return address
80000078:	4422                	lw	s0,8(sp)     # Restore frame pointer
8000007a:	0141                	addi	sp,sp,16    # Deallocate stack
8000007c:	8082                	ret             # Return
```
### String Address Calculation:
- 0x8000008c points to the stored string in memory (not shown in disassembly)
- lui + addi is RISC-V's way to load 32-bit addresses
### _start Entry Point (Address: 0x8000007e)
```asm
8000007e:	1141                	addi	sp,sp,-16    # Allocate stack (unused)
80000080:	c606                	sw	ra,12(sp)     # Save ra (unnecessary)
80000082:	c422                	sw	s0,8(sp)      # Save s0 (unnecessary)
80000084:	0800                	addi	s0,sp,16     # Set frame pointer
80000086:	3fe9                	jal	80000060     # Call main (PC-relative)
80000088:	a001                	j	80000088      # Infinite loop (halt)
```
### Bare-Metal Behavior:
- The j 80000088 creates an infinite loop (replaces OS exit)
- Stack setup is redundant (no caller to return to)
### Key Observations
1. ### Stack Usage:
- Each function creates its own stack frame
- Callee-saved registers (ra, s0) are preserved
2. ### PC-Relative Jumps:
- jal uses relative addressing (3f65 = jump to 0x80000000)
3. ### Inefficiencies:
- Redundant loads/stores due to -O0
- Unnecessary stack ops in _start
4. ### Memory Layout:
- Code starts at 0x80000000 (common RISC-V boot address)
- String likely at 0x8000008c (visible in full objdump)
![Screenshot](Screenshot%20from%202025-06-08%2015-18-10.png)
# Hex Dump & Disassembly
## Show me how to turn my ELF into a raw hex and to disassemble it with objdump. What do each columns mean?
### 1. Convert ELF to Raw Binary (Hex)
```bash
riscv64-unknown-elf-objcopy -O binary hello.elf hello.bin
```
- -O binary: Output format (raw binary)
- Result: hello.bin (contains only executable code/data, no headers)
### To view hexdump:
```bash
hexdump -v -e '4/1 "%02x " "\n"' hello.bin
```
### output:
![Hex Image](./hex.png)
### 2. Disassemble the ELF File
```bash
riscv64-unknown-elf-objdump -d hello.elf
```
### output (with column explanations):
![Hex1 Image](./hex1.png)
### Column Breakdown:
1. ### Address: 80000000
- Memory address where the instruction is loaded.
2. ### Opcode: 7179
- 16-bit compressed instruction (RV32IC). Full 32-bit instructions show 8 hex digits.
3. ### Mnemonic: addi sp,sp,-48
- Human-readable assembly.
- Format: OPCODE DEST, SRC, IMMEDIATE
4. ### Comment: # [1] [2] [3] [4]
- Added manually to explain:
- [1] = Stack pointer adjustment
- [2] = Store operation
- [3] = Frame pointer setup
- [4] = Function body
### 3. Disassemble Raw Binary
```bash
riscv64-unknown-elf-objdump -D -b binary -m riscv:rv32 -M no-aliases hello.bin
```
- -D: Disassemble all sections
- -m riscv:rv32: Target architecture
- -M no-aliases: Show raw instructions (e.g., c.addi instead of addi)
### Output:
![Hex3 Image](./hex3.png)
### 4. Key objdump Flags
| Flag          | Purpose                         |
|------------------|-------------------------------------|
| -d       | Disassemble only code sections   |
| -S        | Mix source with disassembly (requires -g at compile)             |
| -t        | Show symbols  |
| -h  | Section headers         |
| -j .text  | Focus on specific section             |
# ABI & Register Cheat-Sheet
### List all 32 RV32 integer registers with their ABI names and typical calling-convention roles.
### RV32 Integer Registers & Calling Convention
| Register   | ABI Name   | Description                          | Caller/Callee-Saved | Typical Use                     |
|------------|------------|--------------------------------------|----------------------|----------------------------------|
| x0         | zero       | Hardwired zero                       | -                    | Constant 0, discard results      |
| x1         | ra         | Return address                       | Caller               | Stores `jal` return address      |
| x2         | sp         | Stack pointer                        | Callee               | Points to stack                  |
| x3         | gp         | Global pointer                       | -                    | Rarely used global base pointer  |
| x4         | tp         | Thread pointer                       | -                    | Thread-local storage pointer     |
| x5–x7      | t0–t2      | Temporary registers                  | Caller               | Scratch registers                |
| x8         | s0/fp      | Saved register / Frame pointer      | Callee               | Base address of current frame    |
| x9         | s1         | Saved register                       | Callee               | Preserved across calls           |
| x10–x11    | a0–a1      | Function arguments / Return values  | Caller               | Args 1–2, return values          |
| x12–x17    | a2–a7      | Function arguments                  | Caller               | Args 3–8                         |
| x18–x27    | s2–s11     | Saved registers                      | Callee               | Preserved across calls           |
| x28–x31    | t3–t6      | Temporary registers                  | Caller               | Additional scratch registers     |
### Key Calling Convention Rules
1. ### Caller-Saved (Volatile):
- ra, t0-t6, a0-a7
- Can be modified by called functions (save if needed after call).
2. ### Callee-Saved (Non-Volatile):
- sp, s0-s11
- Must be preserved by called functions (save/restore if modified).
3. ### Special Cases:
- a0-a1: Hold return values (only a0 for 32-bit values).
- sp: Must be identical on function exit.
- ra: Contains return address (overwritten by jal).
# Stepping with GDB
### Start QEMU in another terminal with debugging enabled
```bash
qemu-system-riscv32 -nographic -machine sifive_e -kernel hello.elf -S -gdb tcp::1234
```
### Connect GDB to QEMU 
```bash
riscv32-unknown-elf-gdb hello.elf
```
Inside the GDB
```bash
(gdb) target remote :1234
```
Output
```bash
Remote debugging using :1234
0x00001004 in ?? ()
```
### Set breakpoint at main and continue
```bash
(gdb) break main
Breakpoint 1 at 0x1014a: file hello.c, line 4.
(gdb) continue
```
Problem: At this point, QEMU consistently froze after hitting "continue".<br>
Attempting to interrupt using Ctrl+C resulted in:
```bash
Program received signal SIGINT, Interrupt.
0x00000000 in ?? ()
```
Stepping through with stepi stayed stuck at 0x00000000:
```bash
(gdb) stepi
0x00000000 in ?? ()
```
### Output
![Final Output](./finaloutput.png)
![Next Command](nextcommand.png)
# Running Under an Emulator
Run your bare-metal RISC-V ELF program using an emulator like Spike or QEMU, and view the output through the UART console. This is especially useful when real hardware is not available.
### Compile your bare-metal program with debug symbols
Use the following command to compile your C program (hello.c) with the linker script (linker.ld) and include debug info:
```bash
riscv32-unknown-elf-gcc -g -nostdlib -nostartfiles -T linker.ld -o hello.elf hello.c
```
### Run the ELF using QEMU
Use QEMU's RISC-V system emulator to run your ELF and get UART output: 
```bash
qemu-system-riscv32 -nographic -machine sifive_e -kernel hello.elf
```
###  Debugging using GDB with QEMU
Start QEMU with GDB server enabled:
```bash
qemu-system-riscv32 -nographic -machine sifive_e -kernel hello.elf -S -gdb tcp::1234
```
In another terminal, start GDB:
```bash
riscv32-unknown-elf-gdb hello.elf
```
Connect to QEMU's GDB server:
```bash
(gdb) target remote :1234
```
info registers Shows the current values of CPU registers (e.g., ra, sp, gp, a0, etc.)
![emulator](./emulator.png)
disassemble or disassemble Shows the assembly instructions around the program counter or for a specific function
![dump](./dump.png)
# Exploring GCC Optimisation
### Question:
“Compile the same file with -O0 vs -O2. What differences appear in the assembly and why?”
### Goal
To analyze how different GCC optimization levels impact the generated assembly code by comparing the outputs of:
- -O0: No optimization (preserves structure, useful for debugging)
- -O2: Enables aggressive optimizations for performance and efficiency
### Write a test C program
```c
// File: opt.c
int square(int x) {
    return x * x;
}

int unused_function() {
    int y = 100;
    return y;
}

int main() {
    int a = 10;
    int b = square(a);
    return b;
}
```
### Compile with -O0 (no optimization)
```bash
riscv32-unknown-elf-gcc -S -O0 opt.c -o opt_O0.s
```
This generates a verbose .s file with all functions and minimal optimization.
### Step 3: Compile with -O2 (optimized)
```bash
riscv32-unknown-elf-gcc -S -O2 opt.c -o opt_O2.s
```
This will inline small functions, eliminate unused ones, and use registers more efficiently.
### Compare the two versions
```bash
diff opt_O0.s opt_O2.s
```
# 🧪 GCC Optimization Levels: `-O0` vs `-O2`

This project demonstrates how different GCC optimization levels (`-O0` vs `-O2`) affect the generated assembly code in RISC-V or any other target architecture.

---

## ⚙️ What is the difference between `-O0` and `-O2`?

| Flag  | Optimization Level | Description                                                                 |
|--------|--------------------|-----------------------------------------------------------------------------|
| `-O0` | None               | No optimization. Compiler preserves code structure, useful for debugging.   |
| `-O2` | High               | Aggressive optimizations for performance without significantly increasing binary size. |

---

## 🔍 What differences appear in the assembly?

### ✅ With `-O0`:
- The assembly is **longer** and more **verbose**.
- All variables are **stored in memory** (RAM/stack), not registers.
- **No instruction reordering** or **dead code elimination**.
- **Function calls** are not inlined.
- Easier to map each C line to assembly — **good for debugging**.

### ✅ With `-O2`:
- The assembly is **shorter**, **faster**, and more **efficient**.
- **Unused variables** and **redundant operations** are removed.
- **Smarter register allocation** — fewer memory accesses.
- **Loops** may be **unrolled** or **strength-reduced**.
- **Function inlining** replaces small function calls with their code.
- **Instruction scheduling** improves performance through reordering.

---

## 🤔 Why do these differences occur?

The `-O2` flag enables multiple optimization passes in GCC, such as:

- 🔄 **Constant folding** – Compute values at compile-time.
- ✂️ **Dead code elimination** – Remove unused code.
- 🔁 **Loop invariant code motion** – Move static computations out of loops.
- 📦 **Common subexpression elimination** – Avoid redundant calculations.
- ➕ **Inlining** – Replace function calls with actual code.
- 🔧 **Peephole optimization** – Simplify small instruction patterns.

---

## 🧠 Summary

| Aspect             | `-O0`            | `-O2`           |
|--------------------|------------------|-----------------|
| Speed              | Slow             | Fast            |
| Code Size          | Large            | Compact         |
| Debugging          | Easy             | Harder          |
| Optimization       | None             | High            |
| Variable Location  | Mostly memory    | Mostly registers|
| Function Calls     | As-is            | Often inlined   |

---

## 📂 How to Try

```bash
# Compile with no optimization
riscv32-unknown-elf-gcc -O0 -S test.c -o test_O0.s

# Compile with -O2 optimization
riscv32-unknown-elf-gcc -O2 -S test.c -o test_O2.s
```
### My Code
GCC Optimisation 
![GCC Optimisation](./GCC Optimisation.png)
