# 📘 Dedicated Microprocessor for GCD Calculation

This repository presents a **dedicated microprocessor** design for computing the Greatest Common Divisor (GCD) of two 8-bit positive integers, implemented in Logisim. The design follows the **FSM+D** (Finite State Machine + Datapath) methodology described in Hwang's *Digital Logic & Microprocessor Design* fileciteturn0file0.

---

## 🧠 1. Theory & Algorithm

We employ the **Euclidean Algorithm** to compute GCD(X, Y):

```text
1  INPUT X, Y
2  WHILE (X ≠ Y) {
3    IF (X > Y) THEN
4      X = X - Y
5    ELSE
6      Y = Y - X
7  }
8  OUTPUT X   // or Y, since X == Y
```

Key points:

- Two 8-bit registers store X and Y.
- A subtractor executes `X - Y` or `Y - X` based on comparison.
- A comparator generates two status flags: `EQ` (X == Y) and `GT` (X > Y).

---

## 🛠 2. Datapath Design
 ![simulation](./data_path.png)


| Component      | Description                                                               |
| -------------- | ------------------------------------------------------------------------- |
| Registers X, Y | Two 8-bit registers with load control (`XLoad`, `YLoad`)                  |
| Subtractor     | 8-bit subtractor for X-Y and Y-X                                          |
| Multiplexers   | Select inputs for registers and subtractor (signals `In_X`, `In_Y`, `XY`) |
| Comparator     | Produces `EQ` and `GT` flags from X and Y                                 |
| Output Buffer  | Tri-state buffer to drive result on `Out` when `Done` asserted            |

Control signals:

- `In_X`, `In_Y`: choose between primary inputs or subtractor output
- `XLoad`, `YLoad`: load registers X and Y
- `XY`: select subtraction direction
- `Out`: enable output buffer
- `Done`: indicates when computation is complete

---

## 🔄 3. State Machine

### 3.2 Next-State & Output Table

                                       -- Next State (Q₂⁺ Q₁⁺ Q₀⁺) --

| Current State (Q₂ Q₁ Q₀) | EQ,GT = 00 | EQ,GT = 01 | EQ,GT = 10 | EQ,GT = 11 |
|--------------------------|------------|------------|------------|------------|
| 000 (S0)                 | 001        | 001        | 001        | 001        |
| 001 (S1)                 | 011        | 011        | 010        | 100        |
| 010 (S2)                 | 001        | 001        | 001        | 001        |
| 011 (S3)                 | 001        | 001        | 001        | 001        |
| 100 (S4)                 | 100        | 100        | 100        | 100        |
| 101 (Unused)             | 000        | 000        | 000        | 000        |
| 110 (Unused)             | 000        | 000        | 000        | 000        |
| 111 (Unused)             | 000        | 000        | 000        | 000        |



---

## ✍️ 4. Control Logic Equations

### 4.1 Next-State Equations (D-FF Inputs)

```text
D2 = Q2'·EQ + Q2·(¬EQ)        // example Boolean form
D1 = Q1'·(¬EQ·GT) + Q1·(EQ)   // derive via K‑maps
D0 = Q0'·(¬EQ·¬GT) + ...      // complete per K‑map
```

*(Full equations derived from K‑maps in Hwang, Fig. 7.32(c))* fileciteturn0file0

### 4.2 Output (Control) Equations

```text
In_X  = (S == 000)
In_Y  = (S == 000)
XLoad = (S == 000) + (S == 010)
YLoad = (S == 000) + (S == 011)
XY    = (S == 010)
Out   = (S == 100)
Done  = (S == 100)
```

---

## 🔗 5. Complete Control Unit Circuit



- Three D flip-flops for state memory (Q2,Q1,Q0)
- Next-state logic network from above equations
- Output decoders for control signals

---

## 🧪 6. Simulation & Testing

1. **Logisim Project**: `GCD_microprocessor.circ`
2. **Test Vectors**: GCD(12,4) → 4; GCD(15,10) → 5
3. **Waveforms**: see `transient.png` and `ac_sweep.png`

```text
Initial: X=12, Y=4, Reset=1 → State 000 loads inputs.
Clock: FSM cycles through subtraction and comparison until EQ=1.
Final: Out=4, Done asserted.
```

---

## 📂 Repository Structure

```
/README.md
/GCD_microprocessor.circ
/images/
  ├── datapath.png
  ├── control_unit.png
  ├── transient.png
  └── ac_sweep.png
```

---

## 👷 Usage

1. Open the `*.circ` file in Logisim.
2. Apply input via switches and toggle clock.
3. Observe LEDs for output and Done flag.

---

## 📚 References

- E. O. Hwang, *Digital Logic & Microprocessor Design With Interfacing*, 2nd Ed., Cengage Learning, 2018. (Ch. 7, Sec. 7.5.1) fileciteturn0file0

