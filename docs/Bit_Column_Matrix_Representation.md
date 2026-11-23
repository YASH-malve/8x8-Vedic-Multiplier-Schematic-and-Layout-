# Bit-Column Matrix Alignment and Compression Diagrams: Conventional vs. Optimized Vedic Multipliers

#VLSI #BitColumnMatrix #VedicMultiplier #ComputerArithmetic #DotDiagrams

---

## Core Idea: Visualizing Partial Product Matrices as Bit-Weight Columns

In computer arithmetic and digital VLSI design, the most rigorous and intuitive way to analyze multiplier architectures is through **bit-column alignment matrices** (often referred to in IEEE literature as **dot diagrams** or **column-weight matrices**). 

In this representation:
1. **Horizontal Axis (Columns):** Corresponds to binary arithmetic weights $2^k$ (from LSB $2^0$ on the far right to MSB $2^{2N-1}$ on the far left).
2. **Vertical Axis (Rows):** Represents the constituent sub-multiplier words ($q_0, q_1, q_2, q_3$ or $Q_0, Q_1, Q_2, Q_3$) placed at their respective power-of-two positional shifts.
3. **Column Height ($h_k$):** Indicates the exact number of partial product bits that must be summed at weight $2^k$.

```
Bit Column:         7       6       5       4       3       2       1       0
---------------------------------------------------------------------------------
q0[3:0]:                                            q0[3]   q0[2]   q0[1]   q0[0]  --> P[1:0] Direct Wire!
q1[3:0]:                            q1[3]   q1[2]   q1[1]   q1[0]
q2[3:0]:                            q2[3]   q2[2]   q2[1]   q2[0]
q3[3:0]:            q3[3]   q3[2]   q3[1]   q3[0]
---------------------------------------------------------------------------------
Heights:              1       1       3       3       3       3       1       1
```

The fundamental distinction between **conventional** and **optimized** Vedic multipliers is how this matrix is collapsed:
* **The Conventional Approach (Figure 3, Figure 5, Figure 7):** Sums the rows horizontally using **cascading multi-bit ripple adders** (two adders in Stage 1 feeding a third adder in Stage 2), ignoring the column heights and creating long carry-propagation ripple paths ($O(N)$ delay).
* **The Optimized Approach (CSA + CPA):** Operates vertically by **slicing the matrix by column height**:
  * Columns with **Height 1** bypass the adders entirely via **zero-delay direct wires**.
  * Columns with **Height 3** are compressed in parallel by a **single stage of 3:2 Carry-Save Adders (CSA)** in constant time ($O(1)$ delay $\approx 100\,\text{ps}$), reducing the column height from 3 to 2 and generating an additional direct output bit.
  * The resulting height-2 vector pair is merged by a short **Vector Carry-Propagate Adder (CPA)**.

---

## Part I: 2x2 Vedic Multiplier Bit-Column Matrix Analysis

For two 2-bit inputs $\mathbf{A} = [a_1, a_0]_2$ and $\mathbf{B} = [b_1, b_0]_2$, the four partial products $a_i b_j$ align across columns $2^0$ to $2^3$:

```
Bit Column Weight:          2^3         2^2         2^1         2^0
Column Index (k):            3           2           1           0
---------------------------------------------------------------------
Row 0 (b0 partials):                                a1*b0       a0*b0
Row 1 (b1 partials):                    a1*b1       a0*b1
---------------------------------------------------------------------
Raw Column Heights:          1           1           2           1
```

### 1. Conventional Reduction Matrix (Figure 3 Style)
In the conventional architecture shown in your uploaded **Figure 3**, the bits are summed using two cascading Half Adders:

```
Step 1: Direct Wire Bypass:
Column 0:  P[0] = a0 * b0  (Height 1 -> zero logic delay)

Step 2: Half Adder 1 (Column 1):
Inputs:    a1*b0, a0*b1
Outputs:   Sum   s1   ---> P[1] = s1
           Carry c1   ---> Ripples into Column 2!

Step 3: Half Adder 2 (Column 2):
Inputs:    a1*b1, c1  (Wait for c1 from HA1!)
Outputs:   Sum   s2   ---> P[2] = s2
           Carry c2   ---> P[3] = c2
```
* **Bottleneck:** Carry $c_1$ must ripple from HA1 to HA2. Total delay $= t_{\text{AND}} + 2 t_{\text{HA}} \approx 210\,\text{ps}$.

---

### 2. Optimized Reduction Matrix (Figure 3-OPT Style)
Because $a_1 b_1$ and $c_1$ can never simultaneously equal 1 when $a_0 b_0 = 1$, **Half Adder 2 is eliminated mathematically**:

```
Step 1: Column 0 Slice (NAND2 + INV, 6T):
P[0] = a0 * b0

Step 2: Column 1 Slice (AOI22 Compound Gate + INV, 10T):
P[1] = (a1*b0 XOR a0*b1)  (Evaluated directly without carry ripple)

Step 3: Column 2 Slice (NAND3 Compound Gate + INV, 8T):
P[2] = a1 * b1 * (P[0]_bar)  [HA2 eliminated!]

Step 4: Column 3 Slice (Shared NAND2 + INV, 6T):
P[3] = a1 * b1 * P[0]
```
* **Hardware Result:** Transistor count drops from $48\text{T} \to \mathbf{30\text{T}}$ ($-37.5\%$), and critical delay drops from $210\,\text{ps} \to \mathbf{135\,\text{ps}}$ ($35.7\%$ speedup).

---

## Part II: 4x4 Vedic Multiplier Bit-Column Matrix Analysis

### 1. The Raw Partial Product Matrix
Evaluating two 4-bit operands $\mathbf{A}[3:0]$ and $\mathbf{B}[3:0]$ produces four 4-bit sub-products:
$$q_0 = A_{\text{L}} \times B_{\text{L}}, \quad q_1 = A_{\text{H}} \times B_{\text{L}}, \quad q_2 = A_{\text{L}} \times B_{\text{H}}, \quad q_3 = A_{\text{H}} \times B_{\text{H}}$$

Aligning these four sub-multipliers according to their mathematical powers of two:

```
Bit Column Weight:      2^7     2^6     2^5     2^4     2^3     2^2     2^1     2^0
Bit Column Index (k):    7       6       5       4       3       2       1       0
-------------------------------------------------------------------------------------
q0[3:0] (Weight 2^0):                                   q0[3]   q0[2]   q0[1]   q0[0]
q1[3:0] (Weight 2^2):                   q1[3]   q1[2]   q1[1]   q1[0]
q2[3:0] (Weight 2^2):                   q2[3]   q2[2]   q2[1]   q2[0]
q3[3:0] (Weight 2^4):   q3[3]   q3[2]   q3[1]   q3[0]
-------------------------------------------------------------------------------------
Raw Column Heights:      1       1       3       3       3       3       1       1
```

---

### 2. Conventional 3-Adder Reduction Matrix (Figure 5 Style)
In the conventional Figure 5, the matrix is partitioned horizontally into three ripple adders:

```
STAGE 1A: Right Adder (4-bit Ripple Adder across Cols 2 to 5)
Col:                     5       4       3       2
Input 1 (q1[3:0]):      q1[3]   q1[2]   q1[1]   q1[0]
Input 2 ({00, q0[3:2]}):  0       0     q0[3]   q0[2]
-----------------------------------------------------
Right Sum Vector:       c_R4    s_R3    s_R2    s_R1    s_R0  (5 bits)

STAGE 1B: Left Adder (6-bit Ripple Adder across Cols 2 to 7)
Col:                     7       6       5       4       3       2
Input 1 ({q3, 00}):     q3[3]   q3[2]   q3[1]   q3[0]     0       0
Input 2 ({00, q2}):       0       0     q2[3]   q2[2]   q2[1]   q2[0]
---------------------------------------------------------------------
Left Sum Vector:        s_L5    s_L4    s_L3    s_L2    s_L1    s_L0  (6 bits)

STAGE 2: Final Adder (6-bit Ripple Adder across Cols 2 to 7)
Col:                     7       6       5       4       3       2
Left Sum:               s_L5    s_L4    s_L3    s_L2    s_L1    s_L0
Right Sum (aligned):      0     c_R4    s_R3    s_R2    s_R1    s_R0
---------------------------------------------------------------------
Final Sum Q[7:2]:       Q[7]    Q[6]    Q[5]    Q[4]    Q[3]    Q[2]
Direct Wire Q[1:0]:                                             Q[1]    Q[0]
```
* **Critical Path Penalty:** Signals must ripple through Adder Right, then wait and ripple through the Final Adder. Total delay $= 1{,}280\,\text{ps}$ ($640\text{T}$).

---

### 3. Optimized Carry-Save Reduction Matrix (Figure 5-OPT Style)
The optimized architecture groups the matrix vertically by column height:

```
Bit Column Weight:      2^7     2^6     2^5     2^4     2^3     2^2     2^1     2^0
Bit Column Index (k):    7       6       5       4       3       2       1       0
=====================================================================================
STAGE 0: DIRECT WIRE BYPASS (Height 1 Columns)
Direct Extraction:                                                      q0[1]   q0[0]
Output Assignment:                                                      P[1]    P[0]
=====================================================================================
STAGE 1: 4-BIT 3:2 CSA COMPRESSION (Height 3 Columns 2 to 5, Parallel O(1) Delay)
Input Row 1:                            q1[3]   q1[2]   q1[1]   q1[0]
Input Row 2:                            q2[3]   q2[2]   q2[1]   q2[0]
Input Row 3:                            q3[1]   q3[0]   q0[3]   q0[2]
-------------------------------------------------------------------------------------
Compressors:                            CSA_3   CSA_2   CSA_1   CSA_0
-------------------------------------------------------------------------------------
CSA Sum Row (S_csa):                    s[3]    s[2]    s[1]    s[0]   ---> P[2] Direct Wire!
CSA Carry Row (C_csa):          c[3]    c[2]    c[1]    c[0]
Uncompressed Bits:      q3[3]   q3[2]
=====================================================================================
REDUCED MATRIX (Height 2 across Cols 3 to 7)
Row X:                    0     q3[2]   s[3]    s[2]    s[1]   (s[0])   (P[1])  (P[0])
Row Y:                  q3[3]   c[3]    c[2]    c[1]    c[0]
=====================================================================================
STAGE 2: 5-BIT VECTOR CPA MERGE (Short Carry Chain)
Adder Cell Type:         HA2     FA3     FA2     FA1     HA1
Internal Carry Ripple:  cpa[6]  cpa[5]  cpa[4]  cpa[3]   ---
-------------------------------------------------------------------------------------
Final Product Bits:     P[7]    P[6]    P[5]    P[4]    P[3]    P[2]    P[1]    P[0]
```

> [!important] The Direct Wire Breakthrough in Column 2
> In Column 2, Full Adder $\text{CSA}_0$ receives inputs $q_1[0], q_2[0], q_0[2]$.
> Because there is **no carry coming into column 2 from below**, its sum bit $s[0]$ has nothing left to add!
> Therefore:
> $$\mathbf{P[2] = s[0] = q_1[0] \oplus q_2[0] \oplus q_0[2] \quad (\text{Direct Output Pin!})}$$
> This bypasses the entire Stage 2 CPA, reducing the CPA width to only 5 bits (bits 3 to 7)!

---

## Part III: 8x8 Vedic Multiplier Bit-Column Matrix Analysis

### 1. The Raw Partial Product Matrix
Evaluating two 8-bit operands $\mathbf{A}[7:0]$ and $\mathbf{B}[7:0]$ produces four 8-bit sub-products:
$$Q_0 = A_{\text{L}} \times B_{\text{L}}, \quad Q_1 = A_{\text{H}} \times B_{\text{L}}, \quad Q_2 = A_{\text{L}} \times B_{\text{H}}, \quad Q_3 = A_{\text{H}} \times B_{\text{H}}$$

Aligning these four 8-bit sub-multipliers across columns $2^0$ to $2^{15}$:

```
Col Weight:   2^15 2^14 2^13 2^12 2^11 2^10  2^9  2^8  2^7  2^6  2^5  2^4  2^3  2^2  2^1  2^0
Column (k):    15   14   13   12   11   10    9    8    7    6    5    4    3    2    1    0
---------------------------------------------------------------------------------------------
Q0[7:0]:                                                Q0[7]Q0[6]Q0[5]Q0[4] Q0[3]Q0[2]Q0[1]Q0[0]
Q1[7:0]:                          Q1[7]Q1[6]Q1[5]Q1[4] Q1[3]Q1[2]Q1[1]Q1[0]
Q2[7:0]:                          Q2[7]Q2[6]Q2[5]Q2[4] Q2[3]Q2[2]Q2[1]Q2[0]
Q3[7:0]:      Q3[7]Q3[6]Q3[5]Q3[4] Q3[3]Q3[2]Q3[1]Q3[0]
---------------------------------------------------------------------------------------------
Heights:        1    1    1    1    3    3    3    3    3    3    3    3    1    1    1    1
```

---

### 2. Conventional 3-Adder Reduction Matrix (Figure 7 Style)

```
STAGE 1A: Right Adder (8-bit Ripple Adder across Cols 4 to 11)
Inputs:     Q1[7:0] + {0000, Q0[7:4]}
Outputs:    8-bit Sum + Carry out (c_R8)  --> 9 bits total

STAGE 1B: Left Adder (12-bit Ripple Adder across Cols 4 to 15)
Inputs:     {Q3[7:0], 0000} + {0000, Q2[7:0]}
Outputs:    12-bit Sum vector (s_L11 .. s_L0)

STAGE 2: Final Adder (12-bit Ripple Adder across Cols 4 to 15)
Inputs:     Left Sum (12 bits) + Aligned Right Sum (padded to 12 bits)
Outputs:    Q[15:4] (12 bits)
Direct:     Q[3:0] = Q0[3:0] (4 bits)
```
* **Critical Path Penalty:** 32 adders rippling sequentially. Latency $= 2{,}880\,\text{ps}$ ($3{,}456\text{T}$).

---

### 3. Optimized Carry-Save Reduction Matrix (Figure 7-OPT Style)

```
Col Weight:   2^15 2^14 2^13 2^12 2^11 2^10  2^9  2^8  2^7  2^6  2^5  2^4  2^3  2^2  2^1  2^0
Column (k):    15   14   13   12   11   10    9    8    7    6    5    4    3    2    1    0
=============================================================================================
STAGE 0: DIRECT WIRE BYPASS (Height 1 Columns 0 to 3)
Direct Output Extraction:                                               Q0[3]Q0[2]Q0[1]Q0[0]
Primary Outputs:                                                        P[3] P[2] P[1] P[0]
=============================================================================================
STAGE 1: 8-BIT 3:2 CSA COMPRESSION (Height 3 Columns 4 to 11, Parallel O(1) Delay)
Input Row 1:                      Q1[7]Q1[6]Q1[5]Q1[4] Q1[3]Q1[2]Q1[1]Q1[0]
Input Row 2:                      Q2[7]Q2[6]Q2[5]Q2[4] Q2[3]Q2[2]Q2[1]Q2[0]
Input Row 3:                      Q3[3]Q3[2]Q3[1]Q3[0] Q0[7]Q0[6]Q0[5]Q0[4]
---------------------------------------------------------------------------------------------
Compressors:                      CSA7 CSA6 CSA5 CSA4  CSA3 CSA2 CSA1 CSA0
---------------------------------------------------------------------------------------------
Sum Row (S_csa):                  s[7] s[6] s[5] s[4]  s[3] s[2] s[1] s[0] ---> P[4] Direct Wire!
Carry Row (C_csa):           c[7] c[6] c[5] c[4] c[3]  c[2] c[1] c[0]
Uncompressed: Q3[7]Q3[6]Q3[5]Q3[4]
=============================================================================================
REDUCED MATRIX (Height 2 across Cols 5 to 15)
Vector X:       0    0    0  Q3[4] s[7] s[6] s[5] s[4]  s[3] s[2] s[1] (s0)  (Direct P[3:0])
Vector Y:     Q3[7]Q3[6]Q3[5] c[7] c[6] c[5] c[4] c[3]  c[2] c[1] c[0]
=============================================================================================
STAGE 2: 11-BIT VECTOR CPA MERGE (Short Carry Chain)
Adder Type:    HA4  HA3  HA2  FA7  FA6  FA5  FA4  FA3   FA2  FA1  HA1
Carry Ripple: c14  c13  c12  c11  c10   c9   c8   c7    c6   c5   ---
---------------------------------------------------------------------------------------------
Final Product: P15  P14  P13  P12  P11  P10   P9   P8    P7   P6   P5   P4   P3   P2   P1   P0
```

> [!important] Direct Wire Output at Bit 4
> In Column 4, Full Adder $\text{CSA}_0$ receives inputs $Q_1[0], Q_2[0], Q_0[4]$.
> There is **no incoming carry into column 4**.
> Therefore, its sum output $s[0]$ is finalized with zero carry propagation:
> $$\mathbf{P[4] = s[0] = Q_1[0] \oplus Q_2[0] \oplus Q_0[4] \quad (\text{Direct Output Pin!})}$$
> This leaves exactly **5 direct output bits** ($P[4:0]$) finalized before Stage 2 CPA even finishes!

---

## Direct Comparison Table: Conventional vs Optimized Bit-Column Transformations

| Architectural Parameter | $2 \times 2$ Multiplier | | $4 \times 4$ Multiplier | | $8 \times 8$ Multiplier | |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| | **Conv. (Fig 3)** | **Opt. (Fig 3-OPT)** | **Conv. (Fig 5)** | **Opt. (Fig 5-OPT)** | **Conv. (Fig 7)** | **Opt. (Fig 7-OPT)** |
| **Max Column Height** | 2 | **1 (Compound)** | 3 | **3** | 3 | **3** |
| **Stage 1 Mechanism** | HA1 (Ripple) | **AOI Gate Matrix** | 2 Ripple Adders | **Parallel 3:2 CSA** | 2 Ripple Adders | **Parallel 3:2 CSA** |
| **Stage 1 Delay** | $t_{\text{HA}}$ ($100\,\text{ps}$) | **$0\,\text{ps}$ (Merged)** | $t_{\text{RCA}}$ ($440\,\text{ps}$) | **$t_{\text{FA}}$ ($100\,\text{ps}$, $O(1)$)** | $t_{\text{RCA}}$ ($880\,\text{ps}$) | **$t_{\text{FA}}$ ($100\,\text{ps}$, $O(1)$)** |
| **Stage 2 Mechanism** | HA2 (Ripple) | **None (Eliminated)** | Final Ripple Adder | **5-Bit Vector CPA** | Final Ripple Adder | **11-Bit Vector CPA** |
| **Direct Wire Outputs** | $P[0]$ (1 bit) | **$P[0]$ (1 bit)** | $P[1:0]$ (2 bits) | **$P[2:0]$ (3 bits)** | $P[3:0]$ (4 bits) | **$P[4:0]$ (5 bits)** |
| **Total Summation Adders** | 2 Adders | **0 Adders** | 16 Adders | **9 Adders** | 32 Adders | **19 Adders** |
| **Total Transistors** | $48\text{ T}$ | **$30\text{ T}$** | $640\text{ T}$ | **$312\text{ T}$** | $3{,}456\text{ T}$ | **$1{,}656\text{ T}$** |
| **Critical Delay ($t_{\text{pd}}$)** | $210\,\text{ps}$ | **$135\,\text{ps}$** | $1{,}280\,\text{ps}$ | **$535\,\text{ps}$** | $2{,}880\,\text{ps}$ | **$1{,}235\,\text{ps}$** |

```chart
type: bar
labels: [2x2 Conv, 2x2 Opt, 4x4 Conv, 4x4 Opt, 8x8 Conv, 8x8 Opt]
series:
  - title: Propagation Delay (ps)
    data: [210, 135, 1280, 535, 2880, 1235]
    backgroundColor: "#0072B2"
width: 100%
height: 320px
```

---

## Worked Example: Numerical Matrix Propagation ($18 \times 52 = 936$)

Let $\mathbf{A} = 18_{10} = 0001\ 0010_2$ and $\mathbf{B} = 52_{10} = 0011\ 0100_2$.
Four sub-products:
* $Q_0 = 8_{10} = [0, 0, 0, 0, 1, 0, 0, 0]_2$
* $Q_1 = 4_{10} = [0, 0, 0, 0, 0, 1, 0, 0]_2$
* $Q_2 = 6_{10} = [0, 0, 0, 0, 0, 1, 1, 0]_2$
* $Q_3 = 3_{10} = [0, 0, 0, 0, 0, 0, 1, 1]_2$

Populating the actual numerical binary values into the optimized bit-column matrix:

```
Column (k):    15   14   13   12   11   10    9    8    7    6    5    4    3    2    1    0
=============================================================================================
STAGE 0: DIRECT BYPASS
Q0[3:0]:                                                                  1    0    0    0
Outputs:                                                                 P[3] P[2] P[1] P[0] = 0x8
=============================================================================================
STAGE 1: CSA INPUTS (Cols 4 to 11)
Row 1 (Q1):                         0    0    0    0    0    1    0    0
Row 2 (Q2):                         0    0    0    0    0    1    1    0
Row 3 (Q3L/Q0H):                    0    0    1    1    0    0    0    0
---------------------------------------------------------------------------------------------
Compressors:                       CSA7 CSA6 CSA5 CSA4 CSA3 CSA2 CSA1 CSA0
---------------------------------------------------------------------------------------------
Sum Row (S_csa):                    0    0    1    1    0    0    1   [0] ---> P[4] = 0
Carry Row (C_csa):             0    0    0    0    0    1    0    0
Uncompressed (Q3H): 0    0    0    0
=============================================================================================
STAGE 2: CPA VECTOR MERGE (Cols 5 to 15)
Vector X:       0    0    0    0    0    0    1    1    0    0    1   (P[4:0] = 0x08)
Vector Y:       0    0    0    0    0    0    0    0    1    0    0
Adders:        HA4  HA3  HA2  FA7  FA6  FA5  FA4  FA3  FA2  FA1  HA1
---------------------------------------------------------------------------------------------
CPA Sum:        0    0    0    0    0    0    1    1    1    0    1
Outputs:       P15  P14  P13  P12  P11  P10   P9   P8   P7   P6   P5
Hex Nibble:    [------ 0x0 -----]  [------ 0x3 -----]   [--- 0xA ---]  [------ 0x8 -----]
=============================================================================================
Assembled Product: P = 0x03A8 = 936 (Decimal)
```

---

## Traps and Common Pitfalls When Drawing Bit-Column Matrices in Lab Reports

> [!mistake] Misplacing the Carry Output in the Same Column
> In bit-column matrices, remember that a Full Adder at column $k$ outputs:
> * $\text{Sum } s_k$ at column $k$ (same weight $2^k$).
> * $\text{Carry } c_{k+1}$ at column $k+1$ (shifted left by 1 column to weight $2^{k+1}$).
> * Placing the carry in column $k$ will cut the carry magnitude in half and produce erroneous addition in your project report.

> [!warning] Forgetting the MSB Concatenation Shift of Sub-Block $Q_3$
> * $Q_3$ represents $A_{\text{H}} \times B_{\text{H}}$ and possesses an intrinsic positional weight of $2^N$ ($2^4$ at $4 \times 4$, $2^8$ at $8 \times 8$).
> * Its lowest bit $Q_3[0]$ aligns at column $N$, NOT column 0.
> * Always align $Q_3[3:0]$ directly above the high-order bits of $Q_1$ and $Q_2$.

---

## Summary and Reflection

> [!summary] Key Takeaways
> * **Visual Clarity of Bit-Column Matrices:** Column alignment diagrams clearly expose the varying column heights ($1 \to 3 \to 1$) that govern multiplier efficiency.
> * **The Flaw of Conventional Summation:** The conventional 3-adder tree groups the matrix horizontally into arbitrary ripple-carry adders, generating long cascading carry paths ($O(N)$ delay).
> * **The Elegance of Optimized Carry-Save Reduction:**
>   * Height-1 columns bypass all addition logic ($P[1:0]$ at $4 \times 4$, $P[3:0]$ at $8 \times 8$).
>   * Height-3 columns are compressed in a single parallel step ($O(1)$ delay) using 3:2 CSAs, immediately delivering an extra direct bit ($P[2]$ at $4 \times 4$, $P[4]$ at $8 \times 8$).
>   * The remaining height-2 vector pair is resolved by a minimal-length CPA.
> * **Silicon Impact:** Upgrading to the optimized column-slice architecture cuts total transistor count by $52.1\%$ ($3{,}456\text{T} \to 1{,}656\text{T}$) and reduces propagation delay from $2.88\,\text{ns}$ to $1.235\,\text{ns}$ ($2.33\times$ speedup).

> [!think] Design Reflection Questions
> 1. In the $8 \times 8$ matrix, why is the column height bounded at exactly 3 across columns 4 to 11, despite summing four 8-bit sub-multipliers?
> 2. How does the presence of 5 direct output bits ($P[4:0]$) reduce the dynamic switching capacitance and power consumption of the final CPA stage?
