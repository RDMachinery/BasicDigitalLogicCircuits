# Parity Generator / Checker — 74HC86 (XOR)

A **parity generator** computes an extra bit that makes the total number of 1s in a data word either **even** (even parity) or **odd** (odd parity). A **parity checker** verifies the received data matches the expected parity — any single-bit error will be detected.

XOR is the perfect gate for this: chaining XOR gates together effectively **counts** whether the number of HIGH inputs is odd (output = 1) or even (output = 0).

## Chips Required

| Chip | Function | Gates Used |
|------|----------|-----------|
| 74HC86 | Quad 2-input XOR | 3 gates (4-bit parity) |

For an **8-bit** parity generator, a second 74HC86 is needed (7 gates total).

## How It Works

```
XOR truth table reminder:
  0 XOR 0 = 0
  0 XOR 1 = 1
  1 XOR 0 = 1
  1 XOR 1 = 0   ← "are these different?"
```

Chaining XOR gates produces a running "odd parity" flag:

```
P = D0 XOR D1 XOR D2 XOR D3
```

If P = 1 → odd number of 1s in D0–D3  
If P = 0 → even number of 1s in D0–D3

## 4-Bit Parity Generator

### Circuit Diagram

```
D0 ──┐
      ├──[XOR1]── W1 ──┐
D1 ──┘                  ├──[XOR3]── PARITY OUT (even)
                        │
D2 ──┐                  │
      ├──[XOR2]── W2 ──┘
D3 ──┘
```

### Truth Table (Even Parity Output)

| D3 | D2 | D1 | D0 | P (Even) |
|:--:|:--:|:--:|:--:|:---:|
| 0  | 0  | 0  | 0  | 0  |
| 0  | 0  | 0  | 1  | 1  |
| 0  | 0  | 1  | 0  | 1  |
| 0  | 0  | 1  | 1  | 0  |
| 0  | 1  | 0  | 0  | 1  |
| 0  | 1  | 0  | 1  | 0  |
| 0  | 1  | 1  | 0  | 0  |
| 0  | 1  | 1  | 1  | 1  |
| 1  | 0  | 0  | 0  | 1  |
| ... | ...| ...| ...| ...|

P = 1 when an **odd** number of data bits are 1 (even parity bit corrects this)

## Pin Connections — 74HC86 (4-bit, 3 gates used)

```
74HC86 Pinout
  ┌──────────────┐
  │ 1  ──── D0
  │ 2  ──── D1
  │ 3  ──── W1 (D0 XOR D1)
  │           │
  │ 4  ──── D2
  │ 5  ──── D3
  │ 6  ──── W2 (D2 XOR D3)
  │           │
  │ 9  ──── W1 (from pin 3)
  │ 10 ──── W2 (from pin 6)
  │ 8  ──── PARITY OUTPUT
  │           │
  │ 7  ──── GND
  │ 14 ──── VCC
  └──────────────┘
```

## Wiring Summary (4-bit)

```
D0 ──────────────── 74HC86 pin 1
D1 ──────────────── 74HC86 pin 2
                    74HC86 pin 3 (W1) ──┐
                                         │
D2 ──────────────── 74HC86 pin 4         │
D3 ──────────────── 74HC86 pin 5         │
                    74HC86 pin 6 (W2) ──┤
                                         │
                    74HC86 pin 9  ◄───── W1 (pin 3)
                    74HC86 pin 10 ◄───── W2 (pin 6)
                    74HC86 pin 8  ──────► PARITY OUTPUT

VCC (pin 14) ──────── +5V
GND (pin 7)  ──────── GND
```

## Odd Parity

To generate **odd parity** instead of even, simply add an inverter (74HC04) on the output:

```
74HC86 pin 8 ──► 74HC04 (INV) ──► ODD PARITY OUTPUT
```

## 8-Bit Parity Generator (Two 74HC86 chips)

```
Chip 1, Gate 1: D0 XOR D1 ──► W1
Chip 1, Gate 2: D2 XOR D3 ──► W2
Chip 1, Gate 3: D4 XOR D5 ──► W3
Chip 1, Gate 4: D6 XOR D7 ──► W4

Chip 2, Gate 1: W1 XOR W2 ──► X1
Chip 2, Gate 2: W3 XOR W4 ──► X2
Chip 2, Gate 3: X1 XOR X2 ──► PARITY OUTPUT (8-bit)
```

## Using as a Parity Checker

Transmit the data bits **plus** the parity bit. At the receiver, XOR **all** bits together (data + parity):

- Result = **0** → no error detected
- Result = **1** → single-bit error detected

```
Received: D0, D1, D2, D3, P_received

CHECK = D0 XOR D1 XOR D2 XOR D3 XOR P_received

CHECK = 0 → OK
CHECK = 1 → ERROR
```

Wire identically to the generator, but include the received parity bit as a 5th input (requires one extra XOR stage).

## Notes

- Parity can only detect **odd numbers** of bit errors — a 2-bit error cancels out and goes undetected.
- For stronger error detection/correction, see Hamming codes or CRC circuits.
- The gate 4 on the 74HC86 (pins 12, 13, 11) is unused in the 4-bit design — tie inputs to GND.
