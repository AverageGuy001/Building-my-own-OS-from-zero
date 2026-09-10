# Building-my-own-OS-from-zero
# AbsOS

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)]()
[![Target Spec](https://img.shields.io/badge/target-x86--baremetal-orange.svg)]()

A custom bare-metal operating system kernel built from scratch for the `x86` architecture. **AbsOS** aims to provide a lightweight monolithic core covering bootstrapping, low-level hardware interrupt handling, memory management, and device drivers.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Requirements of this projects](#-requirements)
- [Development Environment & Prerequisites](#-development-environment--prerequisites)
- [Getting Started](#-getting-started)
- [Implementation Roadmap](#-implementation-roadmap)
- [License](#-license)

---

## 🎯 Overview

AbsOS is a bare-metal operating system designed to run directly on hardware without relying on host operating system libraries (`-ffreestanding`). 

### Core Goals
* **Freestanding Codebase:** Written strictly in Assembly and C.
* **Bare-Metal Execution:** Running directly on x86 hardware/emulators via QEMU.
* **Core Subsystems:** Modular design separating bootstrapping, memory management, interrupt handling, and device drivers.

---

## 📁 Requirements

In this project we will use:

* **nasm** (Netwide Assembler): Assembler for bootloader/assembly code (low-level).
* **gcc** (GNU Compiler Collection): Compiles C kernel code into object files.
* **binutils** (provides `ld`): The GNU Linker (`ld`) combines object files into a single binary image.
* **qemu-system-x86** (provides `qemu-system-x86_64`): Emulator to test the bootloader and kernel without bare metal.
* **make** (GNU Make): Automates the entire compilation and linking process.

| Tool | Tested Version |
| :--- | :--- |
| **NASM** | 2.15+ |
| **GCC** | 11.0+ |
| **GNU Linker (`ld`)** | 2.38+ |
| **QEMU** | 6.2+ |
| **GNU Make** | 4.3+ |

### Version Check

Run these commands to verify your local versions:

```bash
nasm -v
gcc --version
ld --version
qemu-system-x86_64 --version
make --version
```
To download them copy/paste this command:
- sudo apt update && sudo apt install -y nasm gcc binutils qemu-system-x86 make

