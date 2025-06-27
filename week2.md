# Week-2
## 1.Caravel SoC - Overview of Sub-Modules

The **Caravel SoC**, developed by Efabless, is a System-on-Chip (SoC) harness used in the Open MPW (Multi-Project Wafer) program. It wraps around a **user project** and provides access to various system-level peripherals and interconnects such as SPI, GPIO, and Wishbone.

This repository documents the **four major sub-modules instantiated inside Caravel**:

---

### 🔧 `mgmt_core_wrapper`

- **Purpose**: Wraps the Management SoC Core.
- **Contents**:
  - RISC-V CPU (management core)
  - RAM and bootloader
  - SPI, UART, and GPIO configuration logic
  - Wishbone master interface
- **Function**:
  - Acts as the control core of the chip
  - Manages all system-level configurations
  - Interfaces with the user project via Wishbone and GPIO

---

### 🔧`user_project_wrapper`

- **Purpose**: Holds the user's custom design.
- **Contents**:
  - User logic
  - Wishbone slave interface
  - GPIO interface
- **Function**:
  - The main programmable logic block for user applications
  - Receives configuration and instructions from the management SoC

---

### 🔧`digital_core`

- **Purpose**: Acts as the digital interconnect core of the chip.
- **Contents**:
  - Wiring and interconnection between user and management projects
  - Internal buses (Wishbone)
- **Function**:
  - Routes and manages digital signals
  - Integrates mgmt_core_wrapper, user_project_wrapper, and GPIO blocks

---

### 🔧`housekeeping` (or `housekeeping_spi`)

- **Purpose**: Manages SPI-based system configuration and initial communication.
- **Contents**:
  - SPI interface
  - Control registers
- **Function**:
  - Interfaces with external devices before the RISC-V core boots
  - Enables debugging, configuration, and firmware upload

---

### 📦 Summary

| Sub-Module          | Main Role                                     |
|---------------------|-----------------------------------------------|
| mgmt_core_wrapper   | Controls the chip via RISC-V and system logic |
| user_project_wrapper| Contains user-provided design logic           |
| digital_core        | Manages digital connections and buses         |
| housekeeping_spi    | Enables SPI communication and config setup    |

---

> 📘 This structure makes Caravel a robust and flexible platform for integrating open-source or custom SoC designs for tapeout through the Google/Efabless shuttle program.

## 2.Caravel "Management Protect" Boundary Overview

The **"Management Protect" boundary** in Caravel separates the **Management SoC (`mgmt_core_wrapper`)** from the **User Project Area (`user_project_wrapper`)**.  
It is designed to protect the management area from any faults or issues caused by user logic while enabling controlled communication.

---

### ✅ Signals Crossing the "Management Protect" Boundary

These signals are carefully selected and controlled to maintain both isolation and communication.

---

### 🔹Wishbone Interface Signals

These are used for communication between the management core (as Wishbone master) and the user project (as Wishbone slave):

| Signal Name       | Direction | Description                      |
|-------------------|-----------|----------------------------------|
| `wb_clk_i`        | Input     | Wishbone clock                   |
| `wb_rst_i`        | Input     | Wishbone reset                   |
| `wbs_stb_i`       | Input     | Strobe signal from master        |
| `wbs_cyc_i`       | Input     | Bus cycle signal                 |
| `wbs_we_i`        | Input     | Write enable                     |
| `wbs_sel_i[3:0]`  | Input     | Byte select                      |
| `wbs_dat_i[31:0]` | Input     | Data from master                 |
| `wbs_adr_i[31:0]` | Input     | Address from master              |
| `wbs_ack_o`       | Output    | Acknowledge from user project    |
| `wbs_dat_o[31:0]` | Output    | Data from user project to master |

---

### 🔹Logic Analyzer (LA) Interface

Used for debugging internal user signals via the management core:

| Signal Name           | Direction | Description                            |
|------------------------|-----------|----------------------------------------|
| `la_data_in[127:0]`    | Input     | Data from management to user project   |
| `la_data_out[127:0]`   | Output    | Data from user project to management   |
| `la_oenb[127:0]`       | Input     | Output enable (active low)             |

---

### 🔹GPIO Interface

Shared GPIO pins configured via the management core:

| Signal Name         | Direction | Description                         |
|----------------------|-----------|-------------------------------------|
| `io_in[37:0]`        | Input     | GPIO inputs to user project         |
| `io_out[37:0]`       | Output    | GPIO outputs from user project      |
| `io_oeb[37:0]`       | Output    | Output enable (active low)          |

---

### 🔹Interrupt Lines

Allow the user project to interrupt the management core:

| Signal Name     | Direction | Description                                |
|------------------|-----------|--------------------------------------------|
| `user_irq[2:0]`  | Output    | Interrupts from user to management core    |

---

### 🔹Clock and Reset

Global clock and reset signals passed from management core to user:

| Signal Name | Direction | Description        |
|--------------|-----------|--------------------|
| `clk`        | Input     | System clock       |
| `resetn`     | Input     | Active-low reset   |
##  3.Caravel SoC Clock and Reset Synchronization

In the **Caravel SoC hierarchy**, the **reset and clock signals** originate from external inputs and are propagated internally through structured modules.  
The **first synchronization point** of these signals is **critical** to ensure glitch-free and reliable system operation.

---

### ✅ Where Clock and Reset Are First Synchronized

---

### 🔹 Top-Level: `caravel.v`

The clock and reset signals are received from external chip I/O pads:

| Signal      | Description                           |
|-------------|---------------------------------------|
| `wb_clk_i`  | Wishbone clock input from pad         |
| `wb_rst_i`  | Wishbone reset input (active high)    |

These signals are passed into the `digital_core` module for further propagation.

---

### 🔹 Inside `digital_core.v`

Clock and reset signals are distributed to both:

- `mgmt_core_wrapper` (Management SoC)
- `user_project_wrapper` (User Logic Area)

---

### 📍 Precise Synchronization Point

The **first true synchronization** of reset and clock occurs inside the:

### 🔸 Module: `mgmt_core_wrapper`

Responsibilities of this module include:

- Synchronizing the external reset (`wb_rst_i`) to the system clock
- Generating the internal **glitch-free resetn** signal
- Optionally managing clock gating for power control

---

### 🔧 Reset Synchronization Example

#### Using an async reset with direct assignment:
```verilog
always @(posedge wb_clk_i or posedge wb_rst_i)
  if (wb_rst_i)
    resetn <= 1'b0;
  else
    resetn <= 1'b1;
```

