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

### 1. Data Transfer (SRAM & Registers)



**LDS Rd, K (Load Direct from SRAM)**
* **Description:** Copies a byte from a 16-bit SRAM address into a register.
* **Example:**
    ```assembly
    LDS R16, 0x0100  ; Reads the byte stored at SRAM address 0x0100 
                     ; and copies it into Register R16.
    ```

**STS K, Rr (Store Direct to SRAM)**
* **Description:** Copies a byte from a register into a 16-bit SRAM address.
* **Example:**
    ```assembly
    STS 0x0250, R17  ; Takes the value in R17 and saves it 
                     ; into SRAM address 0x0250.
    ```

**MOV Rd, Rr (Copy Register)**
* **Description:** Copies the value from one register to another.
* **Example:**
    ```assembly
    MOV R1, R16      ; Copies the contents of R16 into R1. 
                     ; R16 remains unchanged.
    ```

---

### 2. I/O Register Operations



**IN Rd, A (In from I/O Location)**
* **Description:** Reads data from an I/O register (like pins or timers) into a CPU register.
* **Example:**
    ```assembly
    IN R20, 0x16     ; Reads the input pins of Port B (PINB) into R20.
                     ; (0x16 is the standard I/O address for PINB).
    ```

**OUT A, Rr (Out to I/O Location)**
* **Description:** Sends data from a CPU register to an I/O register (like turning on an LED).
* **Example:**
    ```assembly
    LDI R18, 0xFF    ; Load 255 (binary 11111111) into R18.
    OUT 0x18, R18    ; Write R18 to Port B (PORTB address 0x18).
                     ; This sets all pins on Port B to HIGH.
    ```

---

### 3. Arithmetic and Logic



**INC Rd (Increment)**
* **Description:** Adds 1 to the value in the register.
* **Example:**
    ```assembly
    LDI R16, 10      ; R16 = 10
    INC R16          ; R16 = 11
    ```

**DEC Rd (Decrement)**
* **Description:** Subtracts 1 from the value in the register.
* **Example:**
    ```assembly
    LDI R17, 5       ; R17 = 5
    DEC R17          ; R17 = 4
    ```

**SUB Rd, Rr (Subtract without Carry)**
* **Description:** Subtracts the source register from the destination register.
* **Example:**
    ```assembly
    LDI R20, 15      ; R20 = 15
    LDI R21, 5       ; R21 = 5
    SUB R20, R21     ; R20 = 15 - 5. Result in R20 is 10.
    ```

**SBC Rd, Rr (Subtract with Carry)**
* **Description:** Subtracts the source register AND the Carry flag from the destination. Used for multi-byte math.
* **Example:**
    ```assembly
    ; Subtracting 16-bit numbers (R17:R16 - R19:R18)
    SUB R16, R18     ; Subtract low bytes
    SBC R17, R19     ; Subtract high bytes with carry from previous step
    ```

**COM Rd (One's Complement)**
* **Description:** Flips all bits in the register (NOT operation).
* **Example:**
    ```assembly
    LDI R16, 0b11110000 ; R16 = 0xF0
    COM R16             ; R16 now becomes 0b00001111 (0x0F)
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
