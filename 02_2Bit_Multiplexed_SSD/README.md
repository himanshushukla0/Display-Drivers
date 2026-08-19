# System Architecture: 2-Bit Time-Division Multiplexed (TDM) Display Driver (`Bit_2_SSD.sch`)

> [!TIP]
> **Overview**: This directory contains the schematic capture, submodule blocks, and simulation files for the **2-Bit / 2-Digit Time-Division Multiplexed Seven-Segment Display Driver** (`Bit_2_SSD.sch`) built in **Xilinx ISE Design Suite**. This system drives two 7-segment display digits over a shared 7-bit segment bus using high-speed scanning and human Persistence of Vision (POV), reducing FPGA pin requirements from 14 pins down to 9 pins ($7\text{ segments} + 2\text{ digit enables}$).

---

## Module Index & File Manifest

| File | Type | Description |
| :--- | :--- | :--- |
| **`Bit_2_SSD.sch`** | Schematic Capture | Top-level system schematic integrating prescaler, dual decoders, and multiplexers |
| **`Frequency_100KHz.sch`** | Schematic Capture | Clock prescaler / divider generating the scanning refresh toggle clock `Q` |
| **`Mux_2_1_7bit.sch`** | Schematic Capture | 7-bit wide 2-to-1 multiplexer routing active segment bus data to `SEG[6:0]` |
| **`Mux_2_1_8bit.sch`** | Schematic Capture | 8-bit wide 2-to-1 multiplexer routing active digit power enables to `AN[7:0]` |
| **`bit_2_ssd_schematic.png`** | Image Artifact | Exported high-resolution schematic capture from Xilinx ISE |

---

## Architectural Block Diagram

```
                                          +---------------------------------+
                    CLK (100MHz) -------->|        Frequency_100KHz         |
                    CLR ----------------->|    (Sequential Clock Divider)   |
                                          +----------------+----------------+
                                                           |
                                                           | Q (Select Signal / Toggle Clock)
                                                           |
        +-------------------------+                        |
IN1 --->|    abcdefg (Upper)      |--- 7-Bit Bus D1 -------+
(4-Bit) |  (Digit 1 BCD Decoder)  |                        |
        +-------------------------+                        v
                                              +--------------------------+
                                              |       Mux_2_1_7bit       |-----> SEG[6:0]
        +-------------------------+           | (7-Bit Segment Selector) |       (Physical Cathodes)
IN0 --->|    abcdefg (Lower)      |--- 7-Bit Bus D0 -------+             |
(4-Bit) |  (Digit 0 BCD Decoder)  |                        ^             |
        +-------------------------+                        |             |
                                                           |             |
                                                           |             |
        +-------------------------+                        |             |
VCC --->|   Pull-Up Array (VCC)   |                        |             |
GND --->|   Pull-Down Array (GND) |------------------------+             |
        +-------------------------+                        v             |
                                              +--------------------------+
                                              |       Mux_2_1_8bit       |-----> AN[7:0] / AN[1:0]
                                              | (8-Bit Digit Anode MUX)  |       (Physical Anodes)
                                              +--------------------------+
```

---

## State Transition & Timing Sequence

$$\begin{array}{c|c|c|c|c|c}
\mathbf{Clock\ Phase\ (Q)} & \mathbf{Active\ Time\ Slot} & \mathbf{Mux\_2\_1\_7bit\ Output} & \mathbf{Mux\_2\_1\_8bit\ Output\ (AN)} & \mathbf{Digit\ 0\ Status} & \mathbf{Digit\ 1\ Status} \\
\hline
\mathbf{Q = 0} & \text{Slot 0 } (0 - 1.0\text{ ms}) & \text{Segments for } IN0 & \mathbf{AN0 = 0\ (ON)}, AN1 = 1\text{ (OFF)} & \mathbf{ILLUMINATED} & \text{DARK} \\
\mathbf{Q = 1} & \text{Slot 1 } (1.0 - 2.0\text{ ms}) & \text{Segments for } IN1 & AN0 = 1\text{ (OFF)}, \mathbf{AN1 = 0\ (ON)} & \text{DARK} & \mathbf{ILLUMINATED} \\
\end{array}$$

---

## Schematic Capture (`Bit_2_SSD.sch`)

![2-Bit Multiplexed SSD Schematic](../assets/bit_2_ssd_sch.png)

---

## Top-Level Verilog Implementation

```verilog
// File: bit_2_ssd_top.v
`timescale 1ns / 1ps

module bit_2_ssd_top (
    input  wire        clk,       // Master Clock
    input  wire        clr,       // Asynchronous Reset
    input  wire [3:0]  in0,       // Digit 0 (Units) BCD
    input  wire [3:0]  in1,       // Digit 1 (Tens) BCD
    output wire [6:0]  seg,       // Shared Segment Bus
    output wire [7:0]  an         // Anode Enable Bus
);

    wire        scan_clk;
    wire [6:0]  seg0, seg1;

    // Prescaler
    freq_prescaler #(.INPUT_FREQ_HZ(100_000_000), .REFRESH_FREQ_HZ(500))
        u_presc (.clk(clk), .rst(clr), .clk_out(scan_clk));

    // Decoders
    bcd_to_7seg u_dec0 (.bcd(in0), .seg(seg0));
    bcd_to_7seg u_dec1 (.bcd(in1), .seg(seg1));

    // Multiplexers
    assign seg = (scan_clk == 1'b0) ? seg0 : seg1;
    assign an  = (scan_clk == 1'b0) ? 8'b1111_1110 : 8'b1111_1101;

endmodule
```
