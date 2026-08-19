# Combinational Logic: BCD-to-7-Segment Decoder (`abcdefg.sch`)

> [!TIP]
> **Overview**: This directory contains the gate-level schematic capture, symbol generation, and truth table specification for the **BCD-to-7-Segment Decoder** (`abcdefg.sch`) designed in **Xilinx ISE Design Suite**. This module accepts a 4-bit Binary Coded Decimal (BCD) input $D[3:0] = \{D_3, D_2, D_1, D_0\}$ and decodes it into the 7 individual segment drive lines ($a, b, c, d, e, f, g$) required to illuminate decimal numerals $0$ through $9$ on a standard Seven-Segment Display (SSD).

---

## Module Index & File Manifest

| File | Type | Description |
| :--- | :--- | :--- |
| **`abcdefg.sch`** | Schematic Capture | Gate-level design with 7 modular segment logic sub-blocks ($a$ through $g$) |
| **`abcdefg.sym`** | Xilinx Symbol | Top-level modular symbol used for hierarchical instantiation in `Bit_2_SSD.sch` |
| **`abcdefg_schematic.png`** | Image Artifact | Exported high-resolution schematic capture from Xilinx ISE |

---

## Truth Table (Active-High & Active-Low)

$$\begin{array}{c|c|cccc|ccccccc|ccccccc}
\mathbf{Digit} & \mathbf{Dec} & \mathbf{D_3} & \mathbf{D_2} & \mathbf{D_1} & \mathbf{D_0} & \mathbf{a_{CC}} & \mathbf{b_{CC}} & \mathbf{c_{CC}} & \mathbf{d_{CC}} & \mathbf{e_{CC}} & \mathbf{f_{CC}} & \mathbf{g_{CC}} & \mathbf{a_{CA}} & \mathbf{b_{CA}} & \mathbf{c_{CA}} & \mathbf{d_{CA}} & \mathbf{e_{CA}} & \mathbf{f_{CA}} & \mathbf{g_{CA}} \\
\hline
0 & 0 & 0 & 0 & 0 & 0 & 1 & 1 & 1 & 1 & 1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 \\
1 & 1 & 0 & 0 & 0 & 1 & 0 & 1 & 1 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 1 & 1 & 1 & 1 \\
2 & 2 & 0 & 0 & 1 & 0 & 1 & 1 & 0 & 1 & 1 & 0 & 1 & 0 & 0 & 1 & 0 & 0 & 1 & 0 \\
3 & 3 & 0 & 0 & 1 & 1 & 1 & 1 & 1 & 1 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 1 & 1 & 0 \\
4 & 4 & 0 & 1 & 0 & 0 & 0 & 1 & 1 & 0 & 0 & 1 & 1 & 1 & 0 & 0 & 1 & 1 & 0 & 0 \\
5 & 5 & 0 & 1 & 0 & 1 & 1 & 0 & 1 & 1 & 0 & 1 & 1 & 0 & 1 & 0 & 0 & 1 & 0 & 0 \\
6 & 6 & 0 & 1 & 1 & 0 & 1 & 0 & 1 & 1 & 1 & 1 & 1 & 0 & 1 & 0 & 0 & 0 & 0 & 0 \\
7 & 7 & 0 & 1 & 1 & 1 & 1 & 1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 1 & 1 & 1 \\
8 & 8 & 1 & 0 & 0 & 0 & 1 & 1 & 1 & 1 & 1 & 1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
9 & 9 & 1 & 0 & 0 & 1 & 1 & 1 & 1 & 1 & 0 & 1 & 1 & 0 & 0 & 0 & 0 & 1 & 0 & 0 \\
\hline
\text{Illegal} & 10..15 & 1 & x & x & x & X & X & X & X & X & X & X & X & X & X & X & X & X & X \\
\end{array}$$

---

## Minimized Boolean SOP Equations

* **Segment a**: $a = D_3 + D_1 + D_2 D_0 + \bar{D}_2 \bar{D}_0$
* **Segment b**: $b = \bar{D}_2 + \bar{D}_1 \bar{D}_0 + D_1 D_0$
* **Segment c**: $c = D_2 + \bar{D}_1 + D_0$
* **Segment d**: $d = D_3 + \bar{D}_2 \bar{D}_0 + D_1 \bar{D}_0 + D_2 \bar{D}_1 D_0 + \bar{D}_2 D_1$
* **Segment e**: $e = \bar{D}_2 \bar{D}_0 + D_1 \bar{D}_0$
* **Segment f**: $f = D_3 + D_2 \bar{D}_1 + D_2 \bar{D}_0 + \bar{D}_1 \bar{D}_0$
* **Segment g**: $g = D_3 + D_2 \bar{D}_1 + \bar{D}_2 D_1 + D_1 \bar{D}_0$

---

## Schematic Capture (`abcdefg.sch`)

![BCD to 7-Segment Decoder Schematic](../assets/abcdefg_sch.png)

---

## Verilog HDL Implementation

```verilog
// File: bcd_to_7seg.v
`timescale 1ns / 1ps

module bcd_to_7seg (
    input  wire [3:0] bcd,
    output reg  [6:0] seg
);

    always @(*) begin
        case (bcd)
            4'h0: seg = 7'b1000000; // 0
            4'h1: seg = 7'b1111001; // 1
            4'h2: seg = 7'b0100100; // 2
            4'h3: seg = 7'b0110000; // 3
            4'h4: seg = 7'b0011001; // 4
            4'h5: seg = 7'b0010010; // 5
            4'h6: seg = 7'b0000010; // 6
            4'h7: seg = 7'b1111000; // 7
            4'h8: seg = 7'b0000000; // 8
            4'h9: seg = 7'b0010000; // 9
            default: seg = 7'b1111111;
        endcase
    end

endmodule
```
