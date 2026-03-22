# SR Latch — 74HC00 (NAND)

An **SR Latch** is the simplest 1-bit memory element in digital logic. It stores a single bit indefinitely until commanded to change. The NAND-based version is the most common implementation and uses just **two cross-coupled NAND gates**.

## Chips Required

| Chip | Function | Gates Used |
|------|----------|-----------|
| 74HC00 | Quad 2-input NAND | 2 gates |

## Truth Table (Active-Low Inputs)

> Note: NAND-based SR latches have **active-low** inputs — a logic 0 activates Set or Reset.

| /S (Set) | /R (Reset) | Q | /Q | State |
|:---:|:---:|:---:|:---:|-------|
| 1 | 1 | Q | /Q | **Hold** (memory) |
| 0 | 1 | 1 | 0 | **Set** |
| 1 | 0 | 0 | 1 | **Reset** |
| 0 | 0 | 1 | 1 | **Forbidden** (avoid!) |

## Circuit Diagram

```
        ┌──────────────────────────────────────────┐
        │                74HC00                    │
        │                                          │
/S ─────┤── pin 1 ┐                                │
        │         ├─[NAND1]── pin 3 ──┬──── Q      │
        │  pin 6 ─┘          (gate1)  │            │
        │    ▲                        │            │
        │    └────────────────────────┘            │
        │                                          │
        │    ┌────────────────────────┐            │
        │    │                        ▼            │
        │  pin 4 ─┐                               │
        │         ├─[NAND2]── pin 6 ──┬──── /Q    │
/R ─────┤── pin 5 ┘          (gate2)  │            │
        │                             │            │
        └─────────────────────────────────────────┘
```

## Pin Connections

### 74HC00 Gate Assignments

```
74HC00
  ┌──────────────┐
  │  1  ──── /S (Set input, active low)
  │  2  ──── feedback from /Q (pin 6)
  │  3  ──── Q output
  │           │
  │  4  ──── feedback from Q (pin 3)
  │  5  ──── /R (Reset input, active low)
  │  6  ──── /Q output
  │           │
  │  7  ──── GND
  │  14 ──── VCC
  └──────────────┘
```

## Wiring Summary

```
74HC00 pin 1  ──── /S (Set, active-low input)
74HC00 pin 2  ──┐
                └── connected to pin 6 (/Q feedback)
74HC00 pin 3  ──── Q output
               └── connected to pin 4

74HC00 pin 4  ──┐
                └── connected to pin 3 (Q feedback)
74HC00 pin 5  ──── /R (Reset, active-low input)
74HC00 pin 6  ──── /Q output
               └── connected to pin 2

VCC (pin 14)  ──── +5V
GND (pin 7)   ──── GND
```

## Cross-Coupling (The Key Connection)

The latch works because each gate's **output feeds back into the other gate's input**:

```
NAND1 output (pin 3 = Q)  ────► NAND2 input (pin 4)
NAND2 output (pin 6 = /Q) ────► NAND1 input (pin 2)
```

This feedback loop is what gives the circuit **memory**.

## Practical Application: Switch Debouncer

Mechanical switches bounce (rapidly open/close) when pressed, causing multiple false triggers. An SR latch debounces cleanly:

```
+VCC
  │
 [R] 10kΩ
  │
  ├──────────── /S (pin 1)
  │
 [Switch]  ← SPDT switch
  │
  ├──────────── /R (pin 5)
  │
 [R] 10kΩ
  │
GND
```

- When the switch moves to /S side: Q goes HIGH immediately, ignoring bounce
- When the switch moves to /R side: Q goes LOW immediately, ignoring bounce

## Notes

- The **forbidden state** (/S = 0 and /R = 0 simultaneously) drives both Q and /Q HIGH, which is contradictory. Avoid this in practice.
- Pull unused inputs HIGH (to VCC via 10kΩ resistor) to avoid floating.
- This circuit is **asynchronous** — it responds immediately to input changes, with no clock.
- Gates 3 and 4 on the 74HC00 are unused — tie their inputs to VCC or GND.
