# Ring Oscillator — 74HC04 (Inverter)

A **ring oscillator** is one of the simplest oscillator circuits possible — it requires no external clock, crystal, or timing capacitor (though one can be added). It works by connecting an **odd number of inverters in a loop**, exploiting propagation delay to generate a continuous square wave.

## Chips Required

| Chip | Function | Gates Used |
|------|----------|-----------|
| 74HC04 | Hex Inverter | 3 or 5 gates |

## How It Works

Each inverter introduces a small **propagation delay** (typically 7–15 ns for 74HC at 5V). With an odd number of inverters in a loop, the signal can never settle — it perpetually chases its own inverted state, producing oscillation.

```
Frequency ≈ 1 / (2 × N × tpd)

Where:
  N   = number of inverter stages
  tpd = propagation delay per gate (~10 ns for 74HC04 at 5V)
```

For a **3-stage** ring oscillator at 5V:
```
f ≈ 1 / (2 × 3 × 10ns) = ~16.7 MHz
```

## Circuit Diagram (3-Stage)

```
        ┌─────────────────────────────────────────────────┐
        │                                                 │
        │   ┌──[INV1]──┬──[INV2]──┬──[INV3]──┐           │
        │   │          │          │          │           │
        └───┘     (observe here)  │          └───────────┘
                                  │
                              OUTPUT
```

## Pin Connections — 74HC04

The 74HC04 contains **6 inverters** in a single 14-pin DIP package:

```
74HC04 Pinout
  ┌────┬────┐
  │ 1  │ 14 │  VCC
  │ 2  │ 13 │
  │ 3  │ 12 │
  │ 4  │ 11 │
  │ 5  │ 10 │
  │ 6  │  9 │
  │ 7  │  8 │
  └────┴────┘
  GND = pin 7

Gate 1: Input = pin 1,  Output = pin 2
Gate 2: Input = pin 3,  Output = pin 4
Gate 3: Input = pin 5,  Output = pin 6
Gate 4: Input = pin 9,  Output = pin 8
Gate 5: Input = pin 11, Output = pin 10
Gate 6: Input = pin 13, Output = pin 12
```

## Wiring Summary (3-Stage Ring)

```
pin 2  (Gate 1 OUT) ────► pin 3  (Gate 2 IN)
pin 4  (Gate 2 OUT) ────► pin 5  (Gate 3 IN)
pin 6  (Gate 3 OUT) ────► pin 1  (Gate 1 IN)  ← feedback loop closes here

OUTPUT  ────────────────── pin 4  (Gate 2 OUT, or any point in the chain)

VCC  (pin 14) ──── +5V
GND  (pin 7)  ──── GND
```

## Slowed-Down Version (RC Ring Oscillator)

To produce a **much lower frequency** (useful for blinking LEDs, etc.), add an RC network between stages:

```
pin 2 ──[R 10kΩ]──┬──► pin 3
                  │
                 [C]  (e.g. 10nF–1µF)
                  │
                 GND
```

Approximate frequency with RC:
```
f ≈ 1 / (2 × N × 0.7 × R × C)
```

Example: N=3, R=10kΩ, C=100nF → f ≈ **~238 Hz**

## Gated Ring Oscillator (using 74HC00 or 74HC08)

Replace one inverter with a **NAND gate** (74HC00) — one input is the oscillator chain, the other is an **ENABLE** signal:

```
ENABLE ──┐
         ├──[NAND]──► replaces INV1 in the ring
FEEDBACK─┘

ENABLE = HIGH → oscillator runs
ENABLE = LOW  → output locked HIGH, oscillator stopped
```

This lets you turn the clock on and off under digital control.

## 5-Stage Ring (Lower Frequency)

For a lower but still fast frequency, use 5 inverters:

```
pin 2 ──► pin 3
pin 4 ──► pin 5
pin 6 ──► pin 9   (jump to gate 4)
pin 8 ──► pin 11  (gate 5)
pin 10 ──► pin 1  (feedback to gate 1)

OUTPUT from pin 4 or any mid-chain gate
```

## Notes

- The 74HC04 oscillates at **RF frequencies** without RC components — you may need a scope to observe it.
- Add a **10–100Ω series resistor** on the output if driving long wires or capacitive loads.
- Unused inverter inputs must be tied to VCC or GND to avoid oscillation on unused gates.
- Supply decoupling is critical at high speeds — place a **100nF ceramic capacitor** between VCC (pin 14) and GND (pin 7), as close to the chip as possible.
- Ring oscillators are sensitive to temperature and supply voltage — not suitable for precision timing.
