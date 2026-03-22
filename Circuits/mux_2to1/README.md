# 2-to-1 Multiplexer — 74HC08 + 74HC04 + 74HC00

A **2-to-1 multiplexer (MUX)** routes one of two input signals to a single output, controlled by a **select line (S)**. It is one of the most useful building blocks in digital logic.

## Chips Required

| Chip | Function | Gates Used |
|------|----------|-----------|
| 74HC08 | Quad 2-input AND | 2 gates |
| 74HC04 | Hex Inverter | 1 gate |
| 74HC00 | Quad 2-input NAND | 1 gate (as OR) |

> The final OR gate is implemented using a **NAND-NAND** equivalence: OR(a,b) = NAND(NOT a, NOT b). Since we already produce intermediate NAND outputs, one NAND gate suffices.

## Logic Expression

```
Y = (A AND NOT S) OR (B AND S)

When S = 0: Y = A
When S = 1: Y = B
```

## Truth Table

| S | A | B | Y (Output) |
|:---:|:---:|:---:|:---:|
| 0 | 0 | X | 0 |
| 0 | 1 | X | 1 |
| 1 | X | 0 | 0 |
| 1 | X | 1 | 1 |

X = don't care

## Circuit Diagram

```
                ┌──────────────────────────────────────────────┐
                │                                              │
S ──────────────┤──[INV]──/S ──┐                              │
                │               ├──[AND1]── A·/S ──┐          │
A ──────────────┤───────────────┘                  ├─[NAND+   │
                │                                  │  NOT]─── Y
B ──────────────┤──────────────┐                  │          │
                │               ├──[AND2]── B·S ───┘          │
S ──────────────┤───────────────┘                              │
                └──────────────────────────────────────────────┘
```

## Detailed Gate-Level Diagram

```
S ────────────► 74HC04 (INV) ────► /S
                                    │
A ──────────────────────────┐       │
                             ├─► 74HC08 (AND1) ────► W1
/S ─────────────────────────┘

B ──────────────────────────┐
                             ├─► 74HC08 (AND2) ────► W2
S ──────────────────────────┘

W1 ─────────────────────────┐
                             ├─► 74HC00 (NAND) ────► W3
W2 ─────────────────────────┘

W3 ─────────────────────────► 74HC04 (INV) ──────► Y (OUTPUT)
```

> W3 = NAND(W1, W2); Y = NOT(W3) = OR(W1, W2) — this is the NAND-as-OR trick.
> 
> Alternatively: use the NAND output directly if your downstream logic can accept active-low.

## Pin Connections

### 74HC04 — Inverter (gates 1 and 2)

```
74HC04
  pin 1  ──── S (select input)
  pin 2  ──── /S (inverted select → to AND1)

  pin 3  ──── W3 (NAND output, from 74HC00 pin 3)
  pin 4  ──── Y (final MUX output)

  pin 7  ──── GND
  pin 14 ──── VCC
```

### 74HC08 — AND Gates (gates 1 and 2)

```
74HC08
  pin 1  ──── A (data input 0)
  pin 2  ──── /S (from 74HC04 pin 2)
  pin 3  ──── W1 (A·/S → to 74HC00 pin 1)

  pin 4  ──── B (data input 1)
  pin 5  ──── S (select input)
  pin 6  ──── W2 (B·S → to 74HC00 pin 2)

  pin 7  ──── GND
  pin 14 ──── VCC
```

### 74HC00 — NAND Gate (gate 1)

```
74HC00
  pin 1  ──── W1 (from 74HC08 pin 3)
  pin 2  ──── W2 (from 74HC08 pin 6)
  pin 3  ──── W3 (NAND output → to 74HC04 pin 3)

  pin 7  ──── GND
  pin 14 ──── VCC
```

## Wiring Summary

```
S  ──────────────────┬──── 74HC04 pin 1  (invert to get /S)
                     └──── 74HC08 pin 5  (AND2 select input)

A  ──────────────────────── 74HC08 pin 1  (AND1 data input)
B  ──────────────────────── 74HC08 pin 4  (AND2 data input)

74HC04 pin 2 (/S) ──────── 74HC08 pin 2  (AND1 select input)

74HC08 pin 3 (W1) ──────── 74HC00 pin 1
74HC08 pin 6 (W2) ──────── 74HC00 pin 2

74HC00 pin 3 (W3) ──────── 74HC04 pin 3  (final inversion)

74HC04 pin 4 ────────────── Y (OUTPUT)

All VCC (pin 14)  ──────── +5V
All GND  (pin 7)  ──────── GND
```

## Notes

- Tie unused gate inputs on all chips to VCC or GND.
- This MUX is combinational — output follows input with only gate propagation delays.
- For a **4-to-1 MUX**, chain two 2-to-1 MUXes together with a second select bit controlling the final stage.
- This circuit uses **3 chips**. In practice, dedicated MUX chips like the **74HC157** (quad 2-to-1 MUX) are more compact, but building it from primitives is excellent for learning.
