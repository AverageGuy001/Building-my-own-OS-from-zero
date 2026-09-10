# Building-my-own-OS-from-zero

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Target](https://img.shields.io/badge/target-x86__64--baremetal-orange.svg)]()
[![Language](https://img.shields.io/badge/language-C%20%7C%20ASM-brightgreen.svg)]()
[![Platform](https://img.shields.io/badge/environment-QEMU-purple.svg)]()

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Requirements of this projects](#-requirements)
- [Theoretical Foundation](#-theoretical-foundation)
- [Getting Started](#-getting-started)
- [Implementation Roadmap](#-implementation-roadmap)
- [License](#-license)

---

## 📌 Overview

This repository documents the step-by-step development of a custom x86 operating system built from scratch. 
Coming into this project with a strong background in C programming but no prior experience in low-level operating system development, my goal is to explore bare-metal architecture, assembly bootloaders, and kernel design hands-on.

### 💡 Transparency & AI Usage

All code and technical documentation in this repository are written and understood by me to ensure high quality and clarity. AI tools are utilized solely to:
* Proofread written documentation for grammar and spelling.
* Verify technical accuracy and spot potential edge cases.
* Serve as an interactive reference for low-level concepts.

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

``` bash
sudo apt update && sudo apt install -y nasm gcc binutils qemu-system-x86 make
```


