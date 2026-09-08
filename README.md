# Building-my-own-OS-from-zero
# AbsOS

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)]()
[![Target Spec](https://img.shields.io/badge/target-x86--baremetal-orange.svg)]()

A custom bare-metal operating system kernel built from scratch for the `x86` architecture. **AbsOS** aims to provide a lightweight monolithic core covering bootstrapping, low-level hardware interrupt handling, memory management, and device drivers.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Project Directory Structure](#-project-directory-structure)
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

## 📁 Project Directory Structure

```text
AbsOS/
├── build/          # Compiled object files and final binary output
├── docs/           # Architecture design notes and reference materials
├── src/            # Kernel assembly and C source code
├── README.md       # Project documentation
└── Makefile        # Build system scripts
