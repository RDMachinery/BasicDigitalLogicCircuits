# Pulse Generator / Edge Detector — 74HC86 + 74HC04

An **edge detector** produces a brief pulse every time an input signal transitions. This circuit exploits the **propagation delay** of an inverter chain to create a narrow pulse on either the rising or falling edge of a signal — a classic technique sometimes called a **glitch generator** or **one-shot pulse**.

## Chips Required

| Chip | Function | Gates Used |
|------|----------|-----------|
| 74HC04 | Hex Inverter | 1–3 gates |
| 74HC86 | Quad 2-input XOR | 1 gate |

## How It Works

1. The input signal is split into two paths.
2. One path passes through an inverter chain, introducing a **propagation delay (tpd)**.
3. Both the delayed and undelayed signals are XORed together.
4. Since XOR outputs HIGH when inputs **differ**, a short pulse appears while the delayed path "catches up" to the direct path.

```
Pulse width ≈ N × tpd

Where:
  N   = number of inverter stages in the delay path
  tpd ≈ 7–15 ns per gate (74HC at 5V)
```

## Circuit Diagram

```
                                ┌──────────────────────────────┐
                                │                              │
                         ┌──── [INV1] ─── [INV2] ─── [INV3] ─┤── pin 2
INPUT ─────────┬─────────┤                                    │   (delayed)
               │         └────────────────────────────────────┘
               │
               └─────────────────────────────────────────────── pin 1
                                                                (direct)
                                                    [XOR]
                                                   pin 3 ──► PULSE OUTPUT
```

## Pulse on Rising and Falling Edge

The XOR sees a difference on **both** rising and falling edges of the input:

```
INPUT:    ____╔════════════╗____
DELAYED:  ____╔══════════════╗____   (slightly late)

XOR OUT:  ____╔╗____________╔╗____
               ▲              ▲
            rising          falling
             edge             edge
             pulse            pulse
```

## Pin Connections

### 74HC04 — Delay Chain (gates 1, 2, 3)

```
74HC04
  pin 1  ──── INPUT signal
  pin 2  ──── delayed by 1× tpd → to XOR (if 1 stage)
  
  For 3-stage delay:
  pin 1  ──── INPUT
  pin 2  ──► pin 3
  pin 4  ──► pin 5
  pin 6  ──── DELAYED OUTPUT → to XOR pin 2

  pin 7  ──── GND
  pin 14 ──── VCC
```

### 74HC86 — XOR Gate (gate 1)

```
74HC86
  pin 1  ──── INPUT (direct, undelayed)
  pin 2  ──── DELAYED (from 74HC04)
  pin 3  ──── PULSE OUTPUT

  pin 7  ──── GND
  pin 14 ──── VCC
```

## Wiring Summary

```
INPUT ──────────────────┬──── 74HC86 pin 1  (direct path)
                        └──── 74HC04 pin 1  (delay path start)

74HC04 pin 2 ──────────────── 74HC04 pin 3  (chain inverters)
74HC04 pin 4 ──────────────── 74HC04 pin 5
74HC04 pin 6 ──────────────── 74HC86 pin 2  (delayed output to XOR)

74HC86 pin 3 ──────────────── PULSE OUTPUT

74HC04 VCC (pin 14) ──────── +5V
74HC04 GND (pin 7)  ──────── GND
74HC86 VCC (pin 14) ──────── +5V
74HC86 GND (pin 7)  ──────── GND
```

## Widening the Pulse with RC

The pulse width from 74HC propagation delays alone is very narrow (tens of nanoseconds). To create a **wider, more visible pulse**, add an RC network to the delay path:

```
INPUT ──[R 1kΩ–100kΩ]──┬──── 74HC04 pin 1 (delay input)
                        │
                       [C]  (1nF–1µF depending on desired width)
                        │
                       GND
```

Approximate pulse width:
```
t ≈ 0.7 × R × C

Examples:
  R=10kΩ, C=10nF  → ~70 µs
  R=10kΩ, C=1µF   → ~7 ms
  R=100kΩ, C=1µF  → ~70 ms
```

> **Note**: Add a **Schmitt trigger** (74HC14) on the 74HC04 input if using RC — slow edges from RC charging can cause double-triggering on plain 74HC04 inputs.

## Rising-Edge-Only Pulse

To detect **only rising edges**, add an AND gate to gate the output with the original signal:

```
PULSE ──┐
         ├──[AND]──► RISING EDGE PULSE ONLY
INPUT ───┘

(the AND suppresses the falling-edge pulse, which occurs when INPUT is LOW)
```

Uses one gate from the **74HC08**.

## Falling-Edge-Only Pulse

Similarly, gate with the **inverted** input:

```
PULSE ──────┐
             ├──[AND]──► FALLING EDGE PULSE ONLY
INPUT──[INV]─┘
```

## Applications

- **Clock edge detection** — trigger other logic on each clock transition
- **Button press pulse** — generate a single clock cycle pulse from a button
- **Delay compensation measurement** — observe gate delays on an oscilloscope
- **Transition counter input** — feed into a register clock input

## Notes

- Pulse width from pure gate delay is in the **nanosecond** range — an oscilloscope is needed to observe it without RC.
- Using an **odd** number of inverters in the delay chain is not required here (unlike a ring oscillator) since the loop is not closed.
- Ensure both the direct and delayed paths reach the XOR at the same PCB trace length to avoid unintended delay mismatch.
