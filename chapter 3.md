# Chapter 3: AVR Microcontroller Architecture & Assembly Guide

Welcome to the Chapter 3 Summary of AVR Microcontroller Architecture and Assembly Language. This guide serves as a quick reference for understanding the core internals of AVR microcontrollers, focusing on memory architecture, general-purpose registers, and fundamental assembly instructions.

---

## Table of Contents
1. [General Purpose Registers (GPRs)](#1-general-purpose-registers-gprs)
2. [Memory Composition](#2-memory-composition)
3. [Data Transfer Instructions](#3-data-transfer-instructions)
4. [Arithmetic and Logic](#4-arithmetic-and-logic)
5. [Status Registers (SREG)](#5-status-registers-sreg)
6. [Assembler Directives](#6-assembler-directives)
7. [Assembling an AVR Program](#7-assembling-an-avr-program)
8. [Architecture Philosophy](#8-architecture-philosophy)

---

## 1. General Purpose Registers (GPRs)

* Most AVR registers are 8-bit registers.
* There are 32 general-purpose registers in AVR (`R0` - `R31`).
* They are located at the very beginning of the memory address space.
* **Crucial Feature:** Registers `R26` to `R31` can be combined into three 16-bit register pairs known as the **X, Y, and Z registers**. These are essential for indirect memory addressing (e.g., iterating through an array of sensor readings).

**Basic Instructions:**
```assembly
LDI Rd, K    ; Load immediate value K into register Rd
             ; Restriction: 16 <= d <= 31 (Only R16-R31 support LDI)
             ; 0 <= K <= 255 (0x00 - 0xFF)

ADD Rd, Rr   ; Add Rr to Rd, store the result in Rd (Rd = Rd + Rr)
```
> **Note:** Use `0x` or `$` to denote hexadecimal values, and `0b` for binary.

---

## 2. Memory Composition

![ATmega32 I/O Registers Overview](assets/images/chapter-3/IO%20Registers%20of%20the%20ATmega32.png)


The AVR memory space is divided into three main data sections:
1. **GPR (General Purpose Registers):** Addresses `0x0000 - 0x001F` (0 - 31). Total: 32 bytes.
2. **I/O Memory:** Addresses `0x0020 - 0x005F` (32 - 95). Total: 64 bytes. This is where peripheral control registers (like Ports, Timers, and ADC) live.
3. **Internal Data SRAM:** Addresses `0x0060 - 0xFFFF`. Used for storing variables and the call stack during runtime.

> **Note:** AVR also features EEPROM, which is non-volatile memory used to save state data (like calibration offsets or user settings) that must persist after the power is turned off.

---

## 3. Data Transfer Instructions

### SRAM & Registers

**LDS Rd, K (Load Direct from SRAM)**
Copies a byte from a 16-bit SRAM address into a register.
```assembly
LDS R16, 0x0100  ; Reads byte at SRAM 0x0100 and copies into R16.
```

**STS K, Rr (Store Direct to SRAM)**
Copies a byte from a register into a 16-bit SRAM address.
```assembly
STS 0x0250, R17  ; Saves value in R17 to SRAM address 0x0250.
```

**MOV Rd, Rr (Copy Register)**
Copies the value from one register to another.
```assembly
MOV R1, R16      ; Copies R16 into R1. R16 remains unchanged.
```

### I/O Register Operations

**IN Rd, A (In from I/O Location)**
Reads data from an I/O register into a CPU register.
```assembly
IN R20, 0x16     ; Reads Port B pins (PINB) into R20.
```

**OUT A, Rr (Out to I/O Location)**
Sends data from a CPU register to an I/O register.
```assembly
LDI R18, 0xFF    ; Load 255 (binary 11111111) into R18.
OUT 0x18, R18    ; Write R18 to Port B (PORTB address 0x18).
                 ; This drives all pins on Port B HIGH.
```

**IN vs LDS Comparison**

| Feature | `IN` Instruction | `LDS` Instruction |
| :--- | :--- | :--- |
| **Speed** | 1 clock cycle (Faster) | 2 clock cycles (Slower) |
| **Size** | 2 Bytes | 4 Bytes |
| **Memory Use** | Less code memory | More code memory |
| **Reach** | Can reach I/O `0x00` -> `0x3F` | Can reach entire memory space |

---

## 4. Arithmetic and Logic

**INC Rd (Increment) & DEC Rd (Decrement)**
Adds or subtracts 1 from the register value. Often used in loop counters.
```assembly
LDI R16, 10      ; R16 = 10
DEC R16          ; R16 = 9
```

**SUB Rd, Rr (Subtract without Carry) & SBC Rd, Rr (Subtract with Carry)**
`SUB` does standard subtraction. `SBC` subtracts the source and the Carry flag, essential for 16-bit (multi-byte) math.
```assembly
; Subtracting 16-bit numbers (R17:R16 - R19:R18)
SUB R16, R18     ; Subtract low bytes
SBC R17, R19     ; Subtract high bytes with carry from previous step
```

**COM Rd (One's Complement) & ANDI Rd, K (Bitwise AND)**
`COM` flips all bits. `ANDI` masks specific bits (useful for isolating a single pin).
```assembly
IN R16, 0x16     ; Read Port B pins
ANDI R16, 0x01   ; Mask all bits except bit 0. 
                 ; R16 will be 1 or 0 depending on pin PB0.
```

---

## 5. Status Registers (SREG)

![Instructions That Affect SREG Flag Bits](assets/images/chapter-3/Instructions%20That%20Affect%20Flag%20Bits.png)

The Status Register (SREG) is an 8-bit flag register that holds information about the result of the most recently executed arithmetic or logic instruction.

| Bit | D7 | D6 | D5 | D4 | D3 | D2 | D1 | D0 |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Flag** | I | T | H | S | V | N | Z | C |

* **I (Global Interrupt):** Enables/disables interrupts globally.
* **T (Bit Copy Storage):** Used with BLD/BST instructions to move single bits.
* **H (Half Carry):** Used for BCD (Binary Coded Decimal) arithmetic.
* **S (Sign Flag):** N XOR V, indicates the true sign of a result.
* **V (Overflow Flag):** Set if 2's complement overflow occurs.
* **N (Negative Flag):** Set if the MSB of the result is 1 (negative result).
* **Z (Zero Flag):** Set to 1 if the result of a math/logic operation is exactly 0.
* **C (Carry Flag):** Set if an operation results in a carry-out (e.g., crossing 255).

---

## 6. Assembler Directives

Directives tell the assembler software what to do; they do *not* generate machine code for the CPU.
```assembly
.EQU LED_PIN = 5 ; Assigns the value 5 to the constant 'LED_PIN'
.ORG 0x0000      ; Put the following instructions starting at memory location 0x0000 
                 ; (Usually the reset vector)
```

---

## 7. Assembling an AVR Program

![Steps to Create an AVR Program](assets/images/chapter-3/Steps%20to%20create%20a%20program.png)


**The Build Flow:**
1. **Editor Program:** You write source code (`myfile.asm`).
2. **Assembler Program:** Converts human-readable assembly into machine code.
3. **Outputs:**
    * `myfile.hex`: The raw machine code flashed to the AVR ROM.
    * `myfile.eep`: EEPROM data (if defined).
    * `myfile.map`: Maps memory addresses to your program labels.
    * `myfile.lst`: A list file combining source code, memory addresses, and hex codes (great for debugging).
    * `myfile.obj`: Object file used for simulation in IDEs.

---

## 8. Architecture Philosophy



* In the ATmega32, the flash memory is 32K bytes (organized as 16K x 16 bits), meaning its Program Counter (PC) is 14 bits wide to address it all.
* **Harvard Architecture:** AVR uses Harvard architecture, meaning it has physically separate memory spaces and buses for **Code (Instructions)** and **Data**.
    * **Program Bus:** 16-bits wide. It fetches instructions quickly because all AVR instructions are 16 or 32 bits.
    * **Data Bus:** 8-bits wide, used to read/write to the SRAM and Registers.
* **RISC (Reduced Instruction Set Computer):** * Fixed instruction sizes (mostly 2 bytes).
    * Executes most instructions in a single clock cycle.
    * Uses a Load/Store architecture (must load data to registers before math).

**Three ways to increase CPU processing power:**
1. Increase clock frequency (causes heat issues).
2. Use Harvard architecture to increase data throughput (implemented by AVR).
3. Use a RISC instruction set to execute one instruction per clock cycle (implemented by AVR).
