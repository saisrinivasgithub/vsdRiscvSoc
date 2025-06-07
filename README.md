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
riscv64 - unknown - elf - gcc - march = rv64imac - mabi = lp64 \
- mcmodel = medany - nostartfiles - ffreestanding \
-Wl ,- Ttext =0 x80000000 -o test . elf test . c
file test . elf
```