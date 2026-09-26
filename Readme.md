
# STM32F4x1CxU6 Black Pill Ultra-Minimalist Assembly 34 bytes LED Blink 🚀

Forget massive HAL libraries, bloated startup files, and hundreds of kilobytes of boilerplate code. This is **bare-metal microprogramming in its purest form**. 

This project achieves a fully working, hardware-validated LED blinking program for the **STM32F401CC** (Black Pill board) written entirely in pure ARM Assembly. No shortcuts, just absolute control over the silicon.

* **Total Binary Size:** Exactly **34 bytes** long (including the vector table)!
* **Target Core:** ARM Cortex-M4

---

## ⚡ The Dirty Hardware Hacks Under the Hood

To squeeze a fully functional program into such a microscopic footprint, standard textbook rules had to be broken. Here is how the compiler and hardware were outsmarted:

### 1. Stack Pointer Hijacking (Zero-Cost RCC Base Address)
Upon a hard reset, the ARM Cortex-M4 core automatically loads the initial Stack Pointer (MSP) from address `0x08000000`. Instead of wastefully pointing it to RAM (since the stack is completely unused here), the MSP is hardcoded to **`0x40023800`** — which is the exact base address of the RCC (Reset and Clock Control) registers.
* **The Payoff:** No need for a bulky 32-bit `LDR` instruction to load the RCC peripheral address. The code uses the `SP` register directly as a pre-loaded pointer to execute `STR R2, [SP, RCC_AHB1_EN_OFFSET]`. Zero bytes wasted!

### 2. Relative Address Math for GPIOC
Instead of loading a brand new 32-bit base address for the GPIOC peripheral (`0x40020800`), the code utilizes a single 16-bit instruction to calculate it relative to the customized Stack Pointer:
* `SUBS R0, SP, 0x3000`
This instantly sets `R0` to point directly to the GPIOC base address using a tiny, ultra-fast subtraction.

### 3. Infinite Loop via Flag Re-use
Instead of an explicit, endless jump instruction (`B`) at the end of the delay loop, the code uses a conditional branch (`BEQ`) hooked to the Zero flag leftover from the decrement routine (`SUBS R1, #1`). Since the delay counter naturally exits when it hits `0`, the branch condition is unconditionally satisfied forever.

---

## 🛠️ How to Verify (The Smart/Lazy Way)

Don't just take my word for it. The best way to inspect this code is to bypass the high-level IDE abstractions and look directly at what the CPU sees.

Run the toolchain disassembler directly on the compiled `.elf` file:

```bash
arm-none-eabi-objdump -d -S startup_stm32f401cdux.elf
```

You will see exactly how the instruction pipeline remains flawlessly tight, keeping the memory layout perfectly packed and sharper than anything a C compiler could ever dream of producing.

---

## 🤷 Why?
Because high-level abstractions have made us soft. This is a reminder of what the hardware can actually do when you talk to it in its own native tongue. 

*Crafted out of pure pragmatism, efficiency, and a healthy dose of engineering laziness.* 😉


![Screenshot](/PICTURES/BLACK_PILL.JPG)
![Screenshot](/PICTURES/ASM.jpg)
