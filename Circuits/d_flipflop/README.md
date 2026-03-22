# Edge-Triggered D Flip-Flop — 74HC00 (NAND)

A **D flip-flop** captures the value of its data input (D) on the rising edge of a clock signal, holding it stable until the next clock edge. It is the fundamental unit of synchronous digital design — used in registers, counters, and state machines.

This implementation uses the **master-slave** architecture built entirely from NAND gates.

## Chips Required

| Chip | Function | Gates Used |
|------|----------|-----------|
| 74HC00 | Quad 2-input NAND | 4 gates (one full chip) |

> A complete edge-triggered D flip-flop requires **6 NAND gates** (two SR latches + a clock edge detector). This design uses a **gated D latch** approach with 4 NAND gates as a simplified version. For a true edge-triggered flip-flop, two 74HC00s are needed.

## Gated D Latch (4 NAND gates — one 74HC00)

The **gated D latch** is transparent when CLK=1 (Q follows D) and holds its value when CLK=0.

### Logic

```
NAND1: output = NAND(D, CLK)      ← /S signal
NAND2: output = NAND(/D, CLK)     ← /R signal  (need inverter or use NAND trick)
NAND3: output = NAND(/S, Q_bar)   ← Q
NAND4: output = NAND(/R, Q)       ← Q_bar
```

### Simplified 4-NAND Version (uses D to derive /D internally)

```
NAND1(D, CLK)    ──────► /S
NAND2(/S, CLK)   ──────► /R   ← This derives NOT(D) using NAND1's output — elegant!
NAND3(/S, Q_bar) ──────► Q
NAND4(/R, Q)     ──────► Q_bar
```

This clever arrangement makes NAND2 act as the inverter and second latch input simultaneously.

## Circuit Diagram

```
                        ┌──────────────────────────────────────┐
                        │             74HC00                   │
                        │                                      │
D ──────────────────────┤── pin 1 ┐                            │
                        │         ├─[NAND1]─ pin 3 ──────────┐│
CLK ────────────────────┤── pin 2 ┘    (/S)            ┌──── ││
                        │               │               │    ││
                        │               └──── pin 4 ─┐  │    ││
                        │                             ├─[NAND2]─ pin 6 (/R)
CLK ────────────────────┤──────────────── pin 5 ─────┘         │
                        │                                       │
                        │  /S ──── pin 9 ┐                      │
                        │                ├─[NAND3]─ pin 8 ─── Q │
                        │  Q_bar ─ pin 10┘                      │
                        │                 ▲                     │
                        │                 │                     │
                        │  /R ──── pin 12─┐                     │
                        │                 ├─[NAND4]─ pin 11 ── Q_bar
                        │  Q ───── pin 13─┘                     │
                        └──────────────────────────────────────┘
```

## Pin Connections — 74HC00

```
74HC00
  ┌──────────────────────────┐
  │ Gate 1 (NAND1)           │
  │  pin 1  ──── D (data in) │
  │  pin 2  ──── CLK         │
  │  pin 3  ──── /S ─────────┼──► pin 9 (NAND3 input)
  │                           │    also ──► pin 4 (NAND2 input)
  │ Gate 2 (NAND2)           │
  │  pin 4  ──── /S (pin 3)  │
  │  pin 5  ──── CLK         │
  │  pin 6  ──── /R ─────────┼──► pin 12 (NAND4 input)
  │                           │
  │ Gate 3 (NAND3) — SR top  │
  │  pin 9  ──── /S (pin 3)  │
  │  pin 10 ──── Q_bar (pin 11)
  │  pin 8  ──── Q output ───┼──► pin 13 (NAND4 input)
  │                           │
  │ Gate 4 (NAND4) — SR bot  │
  │  pin 12 ──── /R (pin 6)  │
  │  pin 13 ──── Q (pin 8)   │
  │  pin 11 ──── Q_bar output┼──► pin 10 (NAND3 input)
  │                           │
  │  pin 7  ──── GND         │
  │  pin 14 ──── VCC         │
  └──────────────────────────┘
```

## Wiring Summary

```
External connections:
  D   ──────────────────── 74HC00 pin 1
  CLK ──────────────┬───── 74HC00 pin 2
                    └───── 74HC00 pin 5

Internal cross-connections:
  pin 3  (/S) ──────┬───── pin 9
                    └───── pin 4

  pin 6  (/R) ───────────── pin 12

  pin 8  (Q)  ──────┬───── OUTPUT Q
                    └───── pin 13

  pin 11 (Q_bar) ───┬───── OUTPUT /Q
                    └───── pin 10

Power:
  pin 14 ──── +5V
  pin 7  ──── GND
```

## Truth Table (Gated D Latch)

| CLK | D | Q (next) | Mode |
|:---:|:---:|:---:|------|
| 0 | X | Q (held) | **Hold** |
| 1 | 0 | 0 | **Reset** |
| 1 | 1 | 1 | **Set** |
| ↑ | D | D | *(edge-triggered version only)* |

## Full Edge-Triggered D Flip-Flop (Two 74HC00 chips)

For a true **rising-edge-triggered** flip-flop, use a **master-slave** configuration — two gated latches wired so the master captures on CLK=0 and the slave transfers on CLK=1:

```
                Master Latch            Slave Latch
D ──► [Gated     ]──── Qm ──► [Gated     ]──── Q
      [D Latch   ]             [D Latch   ]
      [CLK = /CLK]             [CLK = CLK ]──── /Q

CLK ──[INV]──► /CLK
```

- **Master**: enabled when CLK=0 (transparent, samples D)
- **Slave**: enabled when CLK=1 (transfers master output to Q)
- Result: Q only changes on the **rising edge** of CLK

This requires 8 NAND gates total (two 74HC00 packages), plus an inverter (74HC04) for /CLK.

## Notes

- Place a **100nF decoupling capacitor** between VCC and GND, close to the chip.
- Ensure CLK signal edges are fast and clean — slow edges can cause metastability.
- The **setup time** (D must be stable before CLK edge) and **hold time** (D must remain stable after CLK edge) are specified in the 74HC00 datasheet.
- Unused gates: in the 4-gate version, all gates are used. In extended designs, tie unused inputs to VCC or GND.
