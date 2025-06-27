# 🚀 Caravel SoC Overview - Open MPW Project

This repository provides a detailed explanation and support files for working with the [Caravel SoC](https://github.com/efabless/caravel), a harness developed by [efabless](https://efabless.com) that integrates user-defined digital logic with standard system peripherals like SPI, UART, GPIO, and Wishbone.

It is widely used in the **Open MPW Shuttle Program** to help developers submit ASICs using the open-source toolchain.

---

## 📦 Major Components in Caravel

Caravel is structured around **four major sub-modules**:

---

### 🔧 1. `mgmt_core_wrapper`

- Wraps the **Management SoC** core.
- Contains:
  - **RISC-V CPU**
  - **RAM**
  - **Bootloader**
  - Peripheral configuration logic
- Interfaces:
  - **Wishbone Master**
  - **SPI, UART, GPIO configuration**
- Acts as the **main controller** of the chip for booting, peripheral setup, and communication.

---

### 👤 2. `user_project_wrapper`

- Entry point for **user-defined digital designs**.
- Connected to:
  - **Wishbone slave interface** from `mgmt_core_wrapper`
  - **GPIOs**
- Fully customizable:
  - Users instantiate their custom modules here.
  - Logic is inserted in `verilog/rtl/user_project_wrapper.v`.

---

### ⚙️ 3. `digital_core`

- Contains all **digital logic interconnects**.
- Binds together:
  - `mgmt_core_wrapper`
  - `user_project_wrapper`
  - **Wishbone bus**
  - **GPIO cells**
- Routes signals internally and externally (to pads).

---

### 🏠 4. `housekeeping_spi`

- Handles **system-level communication over SPI** before the main CPU boots.
- Key features:
  - Communicates with the chip via SPI for initial configuration.
  - Used to **upload firmware**, debug, or check status registers.
- Critical during **bring-up and debugging** phases.

---

## 🔗 Repository Structure (Example)
caravel/
├── openlane/
│   └── user_project_wrapper/    ← Physical design configs
├── verilog/
│   ├── rtl/
│   │   ├── user_project_wrapper.v  ← Your top module
│   │   └── custom_module.v         ← Your logic
│   └── dv/
│       └── user_project_tb/        ← Testbench files
├── gds/                           ← Final GDSII output
├── def/                           ← DEF layout
└── docs/
    └── images/
        └── caravel_architecture.png


