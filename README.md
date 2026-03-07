### Chapter 3: AVR Microcontroller Architecture & Assembly Summary

#### Section 1: General Purpose Registers (GPRs)
* Most AVR registers are 8-bit registers.
* There are 32 general-purpose registers in AVR (`R0` - `R31`).
* They are located at the very beginning of the memory address space.
* **Crucial Addition:** Registers `R26` to `R31` can be combined into three 16-bit register pairs known as the **X, Y, and Z registers**. These are essential for indirect memory addressing (e.g., iterating through an array of sensor readings).

**Basic Instructions:**
```assembly
LDI Rd, K    ; Load immediate value K into register Rd
             ; Restriction: 16 <= d <= 31 (Only R16-R31 support LDI)
             ; 0 <= K <= 255 (0x00 - 0xFF)

ADD Rd, Rr   ; Add Rr to Rd, store the result in Rd (Rd = Rd + Rr)
