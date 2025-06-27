# Week-2
## Caravel SoC - Overview of Sub-Modules

The **Caravel SoC**, developed by Efabless, is a System-on-Chip (SoC) harness used in the Open MPW (Multi-Project Wafer) program. It wraps around a **user project** and provides access to various system-level peripherals and interconnects such as SPI, GPIO, and Wishbone.

This repository documents the **four major sub-modules instantiated inside Caravel**:

---

## 🔧 1. `mgmt_core_wrapper`

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

## 🔧 2. `user_project_wrapper`

- **Purpose**: Holds the user's custom design.
- **Contents**:
  - User logic
  - Wishbone slave interface
  - GPIO interface
- **Function**:
  - The main programmable logic block for user applications
  - Receives configuration and instructions from the management SoC

---

## 🔧 3. `digital_core`

- **Purpose**: Acts as the digital interconnect core of the chip.
- **Contents**:
  - Wiring and interconnection between user and management projects
  - Internal buses (Wishbone)
- **Function**:
  - Routes and manages digital signals
  - Integrates mgmt_core_wrapper, user_project_wrapper, and GPIO blocks

---

## 🔧 4. `housekeeping` (or `housekeeping_spi`)

- **Purpose**: Manages SPI-based system configuration and initial communication.
- **Contents**:
  - SPI interface
  - Control registers
- **Function**:
  - Interfaces with external devices before the RISC-V core boots
  - Enables debugging, configuration, and firmware upload

---

## 📦 Summary

| Sub-Module          | Main Role                                     |
|---------------------|-----------------------------------------------|
| mgmt_core_wrapper   | Controls the chip via RISC-V and system logic |
| user_project_wrapper| Contains user-provided design logic           |
| digital_core        | Manages digital connections and buses         |
| housekeeping_spi    | Enables SPI communication and config setup    |

---

> 📘 This structure makes Caravel a robust and flexible platform for integrating open-source or custom SoC designs for tapeout through the Google/Efabless shuttle program.



