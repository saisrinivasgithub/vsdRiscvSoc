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

