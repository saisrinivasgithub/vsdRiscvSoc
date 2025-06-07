# vsdRiscvSoc
## How to Install and Unpack the RISC-V Toolchain
installation guide for the RISC-V GNU toolchain targeting RV64 bare-metal systems. This version fixes common build errors and includes essential troubleshooting steps.
# Prerequisites
# Ubuntu
```bash
sudo apt update
sudo apt install autoconf automake autotools-dev curl python3 
```
# Corrected Installation Steps
# 1.Create Workspace
```bash
export WORKSPACE=$HOME/riscv-toolchain
mkdir -p $WORKSPACE
cd $WORKSPACE
```
# 2. Clone Repository with Fixes
```bash
git clone -- depth 1 -- branch 2024.04.05\
https :// github . com / riscv - collab / riscv - gnu - toolchain . git
cd riscv - gnu - toolchain
git submodule update -- init -- recursive -- depth 1 -- jobs 8
```