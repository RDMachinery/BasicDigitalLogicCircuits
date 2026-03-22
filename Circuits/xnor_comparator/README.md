# 1-Bit Comparator (XNOR) — 74HC86 + 74HC04

An **XNOR gate** outputs HIGH when both inputs are **equal**, and LOW when they differ. Since the 74HC family has no dedicated XNOR chip, we build it by inverting the output of a **74HC86 XOR gate**. The result is a simple **1-bit equality comparator**.

```
XNOR(A, B) = NOT(XOR(A, B)) = 1 when A == B
```

## Chips Required

| Chip | Function | Gates Used |
|------|----------|-----------|
| 74HC86 | Quad 2-input XOR | 1 gate |
| 74HC04 | Hex Inverter | 1 gate |

## Truth Table

| A | B | XOR (intermediate) | XNOR / Equal output |
|:---:|:---:|:---:|:---:|
| 0 | 0 | 0 | **1** (equal) |
| 0 | 1 | 1 | **0** (not equal) |
| 1 | 0 | 1 | **0** (not equal) |
| 1 | 1 | 0 | **1** (equal) |

## Circuit Diagram

```
        ┌────────────────────────────────────┐
        │                                    │
A ──────┤── pin 1 ┐                          │
        │         ├─[XOR]─ pin 3 ─[INV]──── EQUAL
B ──────┤── pin 2 ┘  74HC86        74HC04    │
        │                                    │
        └────────────────────────────────────┘
```

## Pin Connections

### 74HC86 — XOR Gate (gate 1)

```
74HC86
  pin 1  ──── A (input)
  pin 2  ──── B (input)
  pin 3  ──── XOR output → to 74HC04 pin 1
  pin 7  ──── GND
  pin 14 ──── VCC
```

### 74HC04 — Inverter (gate 1)

```
74HC04
  pin 1  ──── XOR output (from 74HC86 pin 3)
  pin 2  ──── EQUAL output (A == B when HIGH)
  pin 7  ──── GND
  pin 14 ──── VCC
```

## Wiring Summary

```
A ──────────────────────── 74HC86 pin 1
B ──────────────────────── 74HC86 pin 2

74HC86 pin 3 ─────────────74HC04 pin 1   (XOR → INV)

74HC04 pin 2 ──────────────EQUAL output  (HIGH = A equals B)

74HC86 VCC (pin 14) ──────+5V
74HC86 GND (pin 7)  ──────GND
74HC04 VCC (pin 14) ──────+5V
74HC04 GND (pin 7)  ──────GND
```

## 4-Bit Comparator (Equality Only)

To compare two **4-bit words** (A[3:0] vs B[3:0]), use all four XOR gates on a single 74HC86 to compute per-bit XOR, then AND the four XNOR results together.

### Circuit

```
A0, B0 ──► XOR1 ──► INV ──► EQ0 ──┐
A1, B1 ──► XOR2 ──► INV ──► EQ1 ──┤
A2, B2 ──► XOR3 ──► INV ──► EQ2 ──┼──► AND(EQ0, EQ1, EQ2, EQ3) ──► EQUAL
A3, B3 ──► XOR4 ──► INV ──► EQ3 ──┘
```

All four bits must be equal for the final output to be HIGH.

### Chips for 4-bit version

| Chip | Purpose |
|------|---------|
| 74HC86 | 4× XOR (one chip, all gates used) |
| 74HC04 | 4× INV (one chip, 4 gates used) |
| 74HC08 | AND the four EQ bits (two levels of AND needed) |

### AND tree for 4 inputs

```
EQ0 ──┐
       ├─[AND]── W1 ──┐
EQ1 ──┘               ├─[AND]── EQUAL (4-bit)
                       │
EQ2 ──┐               │
       ├─[AND]── W2 ──┘
EQ3 ──┘
```

Uses 3 AND gates from a single 74HC08.

## Alternative: NAND-Only XNOR

You can build XNOR from **4 NAND gates** alone (no XOR chip needed), using De Morgan identities:

```
XNOR(A,B) = NAND( NAND(A, NAND(A,B)), NAND(B, NAND(A,B)) )
```

```
NAND1: W1 = NAND(A, B)
NAND2: W2 = NAND(A, W1)
NAND3: W3 = NAND(B, W1)
NAND4: Y  = NAND(W2, W3)  ← XNOR output
```

This uses one full **74HC00** chip with no other components needed.

## Notes

- Unused gates on all chips must have inputs tied to VCC or GND.
- For a comparator that also outputs **A > B** and **A < B**, additional logic is needed — consider a dedicated chip like the **74HC85** (4-bit magnitude comparator).
- The XNOR is often described as an **equality detector** — it is the digital equivalent of asking "are these the same?"
