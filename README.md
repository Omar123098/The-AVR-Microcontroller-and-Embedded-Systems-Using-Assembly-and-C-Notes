### Chapter 3 AVR Book Summary

#### Section 1: General Purpose Registers
* Most AVR registers are 8-bit registers.
* There are 32 general-purpose registers in AVR (R0 - R31).
* They are in the lowest location of the memory address.

**Basic Instructions:**
```assembly
LDI Rd, K    ; Load Rd with K
             ; 16 <= d <= 31
             ; 0 <= K <= 255 (0x00 - 0xFF)

ADD Rd, Rr   ; Rd = Rd + Rr
```
*Note: Use `$` or `0x` to denote hex values.*

---

#### Section 2: Memory Composition
The memory is composed of three parts:
1. **GPR (General Purpose Registers):** Addresses `0000 - 001F` (0 - 31), Total: 32 bytes.
2. **I/O Memory:** Addresses `0020 - 005F` (32 - 95), Total: 64 bytes.
3. **Internal Data SRAM:** Addresses `0060 - FFFF`.

*Note: AVR has EEPROM (does not lose its data after power off).*

---

#### Section 3: Data Transfer Instructions

```assembly
LDS Rd, K    ; Load Rd (0 <= d <= 31) from location K
             ; 0000 <= K <= FFFF

STS K, Rr    ; Store Rr to location K
             ; 0000 <= K <= FFFF

IN Rd, A     ; Load an I/O location (A) to Rd
             ; 0 <= A <= 63 , 0 <= d <= 31

OUT A, Rr    ; Store register to I/O

MOV Rd, Rr   ; Rd = Rr

INC Rd       ; Rd = Rd + 1

SUB Rd, Rr   ; Rd = Rd - Rr (without carry)

SBC Rd, Rr   ; Rd = Rd - Rr (with carry)

DEC Rd       ; Rd = Rd - 1

COM Rd       ; Rd's complement (1's complement)
```

**IN vs LDS Comparison**

| Feature | `IN` Instruction | `LDS` Instruction |
| :--- | :--- | :--- |
| **Speed** | 1 cycle | 2 cycles |
| **Size** | 2 Bytes | 4 Bytes |
| **Memory Use** | Less code memory | More code memory |
| **Readability** | Supports I/O names | Memory address |
| **Compatibility** | In all AVRs | Not in all AVRs |
| **Reach** | Can reach `00` -> `3F` | Can reach all memory |

---

#### Section 4: Status Registers 



The Status Register (SREG) is an 8-bit flag register.

| Bit | D7 | D6 | D5 | D4 | D3 | D2 | D1 | D0 |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **SREG** | I | T | H | S | V | N | Z | C |

* **I:** Global Interrupt
* **T:** Bit Copy Storage
* **H:** Half Carry
* **S:** Sign Flag
* **V:** Overflow Flag
* **N:** Negative Flag
* **Z:** Zero Flag
* **C:** Carry Flag

---

#### Section 5: Assembler Directives
```assembly
.EQU C = K   ; Assigns counter value K to name 'C'
.ORG K       ; Put the next instructions in location K in the memory 
             ; (as program counter), place the opcode to mem. loc. 0
```

---

#### Section 6: Intro to AVR Assembly Programming
* `ADD` and `LDI` are instructions that command the CPU.
* `.ORG` and `.EQU` are directives to the assembler.

---

#### Section 7: Assembling an AVR Program
**Flow:**
Editor Program (`myfile.asm`) -> Assembler Program -> Outputs:
* `myfile.eep`
* `myfile.hex`
* `myfile.map`: Shows the labels defined in the program and their values.
* `myfile.lst`: Shows the binary and source code, instructions, memory.
* `myfile.obj`: Used in the Simulator or an Emulator.

---

#### Section 8: Program Counter & Space in AVR 



* In ATmega32, flash is 32K bytes organized as 16K x 16, and its program counter is 14 bits wide.
* In AVR, the internal data bus between the code and ROM is 16-bit wide.
* AVR designers made all instructions 2 or 4 bytes; this is part of the RISC architecture philosophy.
* **Harvard Architecture:** AVR uses Harvard architecture, which means there are separated buses for code and data memory.
    * **Program Bus:** The data bus = address bus = PC register = 16-bit to enable the CPU to address the entire program flash ROM.
    * **Data Bus:** The data bus is 8-bits wide.

---

#### Section 9: RISC Architecture
**Three ways to increase the processing power of the CPU:**
1. Increase the clock frequency (but it causes a heat problem).
2. Use Harvard architecture by increasing the number of buses (but in some microprocessors, this is very expensive).
3. Use RISC architecture.

*Note: Atmel uses all three methods.*

**RISC vs CISC details:**
* RISC processors have a fixed instruction size.
* CISC instructions can be 1, 2, or 3 bytes.
* RISC is designed to be friendly with the C language.
* RISC uses a load/store architecture, whereas in CISC microprocessors, data can be manipulated while it is still in memory.

---

#### Section 10: Viewing registers and memory using AVR Studio
*(This section title was crossed out in the notes with no further text.)*
