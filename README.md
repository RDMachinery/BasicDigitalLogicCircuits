# 74HC Logic Gate Circuits

A collection of classic digital logic circuits built entirely from 74HC series logic gate ICs — no microcontrollers, no programmable logic. Just gates.

Each circuit uses some combination of four fundamental chips:

| Chip | Function |
|------|----------|
| **74HC00** | Quad 2-input NAND |
| **74HC04** | Hex Inverter (NOT) |
| **74HC08** | Quad 2-input AND |
| **74HC86** | Quad 2-input XOR |

---

## Circuits

### 1. Half Adder
**Chips:** 74HC86 · 74HC08

Adds two single bits together, producing a **Sum** and a **Carry** output. The XOR gate computes the sum (1+1 wraps to 0), while the AND gate detects the carry (1+1 = 10 in binary). It is the foundational building block of all binary arithmetic and can be chained into a full adder by combining two half adders together.

---

### 2. SR Latch
**Chips:** 74HC00

The simplest possible **1-bit memory cell**, built from just two cross-coupled NAND gates. Set the latch HIGH with the S input, reset it LOW with the R input, and it holds its state indefinitely when both inputs are released. Immediately useful as a **switch debouncer** — a mechanical switch wired to an SR latch produces a clean, bounce-free digital signal.

---

### 3. Ring Oscillator
**Chips:** 74HC04

An odd number of inverters connected in a loop will oscillate forever, because the signal can never settle to a stable state. The oscillation frequency is determined by the propagation delay of the gates — at 5V, a 3-stage 74HC ring oscillator runs at approximately **16 MHz**. Adding an RC network between stages slows it down to audio or visible frequencies, making it useful for blinking LEDs or generating clock signals.

---

### 4. 2-to-1 Multiplexer
**Chips:** 74HC08 · 74HC04 · 74HC00

Routes one of two input signals to a single output, chosen by a **select line**. When the select is LOW, input A passes through; when HIGH, input B does. Built from AND gates to mask each input, an inverter to generate the complementary select, and a NAND used as an OR to combine the two masked signals. Essential for signal routing in larger digital systems.

---

### 5. Parity Generator / Checker
**Chips:** 74HC86

Chaining XOR gates together counts whether the number of HIGH bits in a word is odd or even. The output — a single **parity bit** — is appended to a data word before transmission. At the receiver, XORing the data with the parity bit will equal zero if no error occurred, or one if a single bit was flipped in transit. A simple and elegant form of error detection used throughout digital communications.

---

### 6. D Flip-Flop (Gated D Latch)
**Chips:** 74HC00

Uses all four NAND gates on a single 74HC00 to build a **gated D latch** — a clocked 1-bit memory element. When the clock (enable) is HIGH the output follows the data input; when the clock goes LOW the last value is latched and held. Two of these latches wired as a master-slave pair form a true **edge-triggered D flip-flop**, the core building block of registers, counters, and synchronous state machines.

---

### 7. XNOR / 1-Bit Comparator
**Chips:** 74HC86 · 74HC04

An XNOR gate outputs HIGH when both inputs are **equal** and LOW when they differ, making it a natural 1-bit equality comparator. Since the 74HC family has no dedicated XNOR chip, one is constructed by inverting the output of a 74HC86 XOR gate. Four of these stages, combined with an AND tree (74HC08), extend the design to a **4-bit equality comparator** using a single chip of each type.

---

### 8. Pulse Generator / Edge Detector
**Chips:** 74HC86 · 74HC04

Produces a short pulse every time an input signal changes state. The input is split into two paths — one direct, one delayed through an inverter chain. XORing the two paths together yields a pulse during the brief window when the delayed copy differs from the original. Without RC components the pulse is just nanoseconds wide (visible only on a scope); adding an RC network to the delay path widens the pulse to any desired duration, from microseconds to hundreds of milliseconds.

---

## Chip Summary

| Circuit | 74HC00 | 74HC04 | 74HC08 | 74HC86 |
|---------|:------:|:------:|:------:|:------:|
| Half Adder | | | ✓ | ✓ |
| SR Latch | ✓ | | | |
| Ring Oscillator | | ✓ | | |
| 2-to-1 Multiplexer | ✓ | ✓ | ✓ | |
| Parity Generator | | | | ✓ |
| D Flip-Flop | ✓ | | | |
| XNOR Comparator | | ✓ | | ✓ |
| Pulse Generator | | ✓ | | ✓ |

---

## General Notes

- All chips operate on **2V to 6V**; 5V is the typical supply for 74HC logic.
- Place a **100nF ceramic decoupling capacitor** between VCC and GND on each chip, as close to the package as possible.
- **Tie unused gate inputs** to VCC or GND — never leave them floating, as this causes unpredictable behaviour and increases power consumption.
- Each chip in the 74HC family contains **multiple gates** in one package. Most circuits here use only a fraction of a chip's gates, leaving spares available for other purposes on the same board.

---

## Further Reading

Each circuit has its own dedicated README with full truth tables, pin-by-pin wiring diagrams, and extension ideas:

- [Half Adder](half_adder/README.md)
- [SR Latch](sr_latch/README.md)
- [Ring Oscillator](ring_oscillator/README.md)
- [2-to-1 Multiplexer](mux_2to1/README.md)
- [Parity Generator / Checker](parity_generator/README.md)
- [D Flip-Flop](d_flipflop/README.md)
- [XNOR / 1-Bit Comparator](xnor_comparator/README.md)
- [Pulse Generator / Edge Detector](pulse_generator/README.md)
