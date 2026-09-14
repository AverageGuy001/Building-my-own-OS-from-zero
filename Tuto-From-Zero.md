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
## 🧠 Theoretical Foundation:

> ⚠️ **Prerequisite:** Please read the **Theoretical Foundation** section before exploring the code. The practical implementation relies on key architectural concepts that are explained there first.

### 1. The Compilation & Execution Pipeline:

[ C Code ] ──► Compiler ──► [ Assembly ] ──► Assembler ──► [ Object File ] ──► Linker ──► [ Executable ] ──► [ CPU ]

> Explanation:
We start with C code written by humans. Since the processor cannot understand high-level code directly, we use a Compiler (gcc) to translate the C logic into human-readable low-level Assembly code. Next, the Assembler (nasm) converts that assembly code into low-level machine code (binary instructions) stored in an Object File (.o).However, an object file is still incomplete—it contains unresolved memory addresses and function references. Finally, the Linker (ld) combines all object files, resolves their memory layouts using a linker script, and outputs a complete executable ready file for the CPU to load and run.


### 2. Memory System

Every program and kernel executes inside system RAM, which is divided into specific functional segments. Because CPU registers have very limited storage, they act as active pointers tracking key locations inside this RAM layout:

```text
┌─────────────────────────────────────────────────────────┐
│                    RAM (SYSTEM MEMORY)                  │
├─────────────────────────────────────────────────────────┤
│ High Address 0xFFFFFFFF                                 │
│ ┌─────────────────────────────────────────────────────┐ │
│ │ Stack (Grows Downward ↓)  ◄── [ ESP / RSP Register ]│ │
│ │ Local variables, function frames                    │ │
│ │ ↓                                                   │ │
│ │                                                     │ │
│ │ ↑                                                   │ │
│ │ Heap (Grows Upward ↑) malloc()                      │ │
│ ├─────────────────────────────────────────────────────┤ │
│ │ .bss  (Uninitialized Global / Static Variables)     │ │
│ ├─────────────────────────────────────────────────────┤ │
│ │ .data (Initialized Global / Static Variables)       │ │
│ ├─────────────────────────────────────────────────────┤ │
│ │ .rodata (Read-Only Data, e.g., string literals)     │ │
│ ├─────────────────────────────────────────────────────┤ │
│ │ .text (Compiled Machine Code Instructions)          │ │
│ └─────────────────────────────────────────────────────┘ │
│ Low Address  0x00000000     ◄── [ EIP / RIP Register ]│ │
└─────────────────────────────────────────────────────────┘
```

While RAM stores bulk data, CPU registers perform high-speed operations. Key registers include:
- RAX : holds the return value of a function.
- RDI , RSI , RDX , RCX , R8 , R9 : function arguments RDI the first arg and so on.
- RIP : point to the next instruction.
- RSP : store the memory of the top of the stack.

### 👨‍👦 x86 Register Family Hierarchy

In x86 architecture, smaller register names are simply lower-bit views into the exact same physical storage slot on the CPU silicon chip.

| 64-bit (Father) | 32-bit (Son) | 16-bit (Grandson) | 8-bit (Great-Grandson) |
| :--- | :--- | :--- | :--- |
| `RAX` | `EAX` | `AX` | `AL` |
| `RBX` | `EBX` | `BX` | `BL` |
| `RCX` | `ECX` | `CX` | `CL` |
| `RDX` | `EDX` | `DX` | `DL` |
| `RDI` | `EDI` | `DI` | `DIL` |
| `RSI` | `ESI` | `SI` | `SIL` |
| `RSP` | `ESP` | `SP` | `SPL` |
| `RIP` | `EIP` | `IP` | — |


Here is an example where i will show u the C code and his assembly version:
```bash
  int add(int a, int b) {
      return a + b;
  }
-------------------------------
  mov eax,edi
  add eax,esi
  ret
```
the first section is an easy function that returns the sum of two integers (we are working here with 32 bits) , the next section we can see that:
- First line is equivalent of EAX = EDI
- Second line is equivalent of EAX = EAX + ESI
- Third line is equivalent of returning the value of EAX

->So if we call that function add(5,3) then :
- EDI = 5 , ESI = 3 , EAX = 5+3 = 8

### Linker / Linker scripts:
We already know that the linker is responsible for combining all object files (.o) into a single executable or binary image. In bare-metal development, to dictate the exact RAM addresses for each section (.text, .data, .bss) and define the entry point where execution begins, we write a linker script.

> Bare-metal dev : is writing software directly on the hardware no OS based

```text
       Object Files (.o)             Linker Script (link.ld)
  ┌─────────────────────────┐      ┌─────────────────────────┐
  │ boot.o   (_start)       │      │ ENTRY(_start)           │
  │ kernel.o (code, data)   │ ──►  │ . = 0x100000;           │
  │ main.o   (code, data)   │      │ .text .rodata .data .bss│
  └─────────────────────────┘      └─────────────────────────┘
                │                               │
                └───────────────┬───────────────┘
                                ▼
                         [ LINKER (ld) ]
                                │ 
                                ▼
                    Final Kernel Binary (kernel.bin)
```
-> The linker takes the object files (such as kernel.o and main.o) and merges them into a single executable file (kernel.bin). To achieve this, it performs two core tasks:
- Symbol Resolution: Connects function calls and variable references across separate files so every symbol points to a valid memory location.
- Relocation: Combines identical sections (merging .text from all files into one .text block) and patch-fixes memory jump addresses to match their actual physical RAM locations.

### Linker script : 

```nasm
ENTRY(_start)
SECTIONS {
  . = 0x100000;
  .text : {
    *(.text)
  }
  .data : {
    *(.data)
  }
  .rodata : {
    *(.rodata)
  }
  .bss : {
    *(.bss)
  }
}
```
-> This is an example of a linker script and here is what each line means :
- ```ENTRY(_start)```: tells the linker that execution start with _start (is technically a symbol or label (usually defined in assembly as global _start)).
- ```. = 0x100000```: In linker scripts, . is called the location counter in other words like setting a cursor into starting adress.
- ```*(.text)```: the "*" is to say all and .text means the section it self so it merge all the .text section into one block.

### Execution Environments : 
According to the ISO C Standard, C execution environments are categorized into two distinct **Implementation Conformance Modes** depending on whether an underlying operating system is present:

| Feature | Hosted C Environment | Freestanding C Environment (Bare Metal) |
| :--- | :--- | :--- |
| **Execution Layer** | Runs on top of an OS (Linux, Windows, macOS) | Runs directly on bare hardware / silicon |
| **Program Entry Point** | OS C-runtime calls `int main(...)` | Hardware/Bootloader jumps to `_start` |
| **C Standard Library** | Complete standard library (`stdio.h`, `stdlib.h`, `math.h`) | Minimal type-only headers (`stdint.h`, `stddef.h`, `stdbool.h`) |
| **System Calls & I/O** | Handled by OS (`printf()`, `malloc()`, `write()`) | **None** (Must write custom MMIO/driver code) |
| **Memory Management** | OS kernel handles virtual memory & heap allocation | Developer configures page tables, GDT, & physical allocators |
| **Linker Script** | Default OS linker script (hidden) | Custom Linker Script (`link.ld`) setting physical addresses |
| **Compiler Flags** | Default GCC behavior | Requires `-ffreestanding -nostdlib -fno-builtin` |

In summary, a **Freestanding C Environment** operates on bare metal without OS support or built-in standard library functions, whereas a **Hosted C Environment** relies on an underlying operating system to provide full standard library.

### 3. Assembly Essentials :

i will represent some basic commands that casually you will meet them:

```nasm
### Data Transfer & Arithmetic:

mov dest, src           ; dest = src
lea rax, [rbx + 8]      ; RAX = RBX + 8 (Calculates effective memory address without dereferencing)
mov rax, [rbx + 8]      ; RAX = Memory[RBX + 8] (Dereferences pointer to fetch stored value)
add a, b                ; a = a + b
sub a, b                ; a = a - b
[RAX + i * 4]           ; Array index offset: arr[i] where element size is 4 bytes (int32)

### Comparaison & Logic:

cmp a, b                ; Computes (a - b) to set CPU flags (does NOT modify register 'a')
test a, a               ; Computes bitwise (a & a) to check if 'a' is zero or negative

### Control Flow & Conditionals:

jmp label               ; Unconditional jump to target address
je  label               ; Jump if Equal 
jne label               ; Jump if Not Equal 
jg  label               ; Jump if Greater
jl  label               ; Jump if Less

-> for example:
section .text
global _start

_start:
    mov eax, 10         ; EAX = 10
    
    cmp eax, 10         ; Compare EAX with 10 (computes EAX - 10)
    je  .is_equal       ; Jump to '.is_equal' if Zero Flag (ZF) is set (i.e., EAX == 10)

.is_not_equal:
    mov ebx, 0          ; Return status code 0 (False)
    jmp .exit

.is_equal:
    mov ebx, 1          ; Return status code 1 (True)

.exit:
    mov eax, 1          ; Linux sys_exit system call number
    int 0x80            ; Call Linux kernel interrupt
```
Ignore the other part that are new and focus u see the value of eax = 10 then direclty jump to the lable of the equal case and follow the instructions at the end we got ebx = 1 and eax = 10.

- Push vs Pop :
   ```nasm
   push rax
   pop rax
   ```
  -> push rax: Decrements RSP by 8 bytes (RSP = RSP - 8) and writes the value of RAX to [RSP].
  -> pop rax: Reads the value at [RSP] into RAX and increments RSP by 8 bytes (RSP = RSP + 8).
  ```text
  BEFORE PUSH                    AFTER PUSH                    AFTER POP
  ┌──────────────────┐          ┌──────────────────┐          ┌──────────────────┐
  │ [Unused Stack]   │          │ [Unused Stack]   │          │ [Unused Stack]   │
  ├──────────────────┤          ├──────────────────┤          ├──────────────────┤
  │ 0x000000000000   │ ◄─ RSP   │ 0x000000000000   │          │ 0x000000000000   │ ◄─ RSP
  ├──────────────────┤          ├──────────────────┤          ├──────────────────┤
  │                  │          │ Value of RAX     │ ◄─ RSP   │ [Popped/Garbage] │
  └──────────────────┘          └──────────────────┘          └──────────────────┘
   (High Addresses)               (Low Addresses ↓)            (RSP restored ↑)
  ```
  
