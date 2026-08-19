# Digital Logic Design: Seven-Segment Display (SSD) & 2-Bit Multiplexed Display Drivers

<div align="center">

[![Xilinx ISE](https://img.shields.io/badge/EDA-Xilinx%20ISE%2014.7-blue.svg)](https://www.xilinx.com/)
[![Simulation](https://img.shields.io/badge/Simulation-ISim%20Behavioral-success.svg)](https://www.xilinx.com/)
[![Language](https://img.shields.io/badge/HDL-Verilog%20%7C%20Schematic%20Capture-orange.svg)](https://www.xilinx.com/)
[![FPGA Targets](https://img.shields.io/badge/FPGA-Spartan--3A%20%7C%20Spartan--6%20%7C%20Artix--7-purple.svg)](https://www.xilinx.com/)
[![Design Pattern](https://img.shields.io/badge/Architecture-Time--Division%20Multiplexing%20(TDM)-brightgreen.svg)](#2-bit-2-digit-ssd-architecture-bit_2_ssdsch)

</div>

> [!TIP]
> **Repository Overview**: This repository delivers an industrial-grade, gate-level and behavioral digital architecture for **Seven-Segment Display (SSD) Interfaces** and **2-Bit / Multi-Digit Time-Division Multiplexed (TDM) Display Drivers**. Developed and verified within the **Xilinx ISE Design Suite (vP.20131013)** for Xilinx Spartan and Artix FPGA families, this project bridges fundamental sequential logic (Flip-Flops, Shift Registers) and digital counters (MOD-10 BCD counters) with real-world optoelectronic human-machine interfaces.

---

## Table of Contents

1. [Architectural Continuum: From Sequential Registers to Display Drivers](#architectural-continuum-from-sequential-registers-to-display-drivers)
2. [Fundamentals of Seven-Segment Displays (SSDs)](#fundamentals-of-seven-segment-displays-ssds)
   * [Segment Topologies & Naming Conventions](#segment-topologies--naming-conventions)
   * [Common Anode (CA) vs. Common Cathode (CC) Configurations](#common-anode-ca-vs-common-cathode-cc-configurations)
   * [Electrical Characteristics & Resistor Sizing Calculations](#electrical-characteristics--resistor-sizing-calculations)
3. [Single-Digit BCD-to-7-Segment Decoder (`abcdefg.sch`)](#single-digit-bcd-to-7-segment-decoder-abcdefgsch)
   * [Functional Specification & Truth Table](#functional-specification--truth-table)
   * [K-Map Derivations for Segments a through g](#k-map-derivations-for-segments-a-through-g)
   * [Schematic Architecture & Gate Decomposition (`abcdefg.sch`)](#schematic-architecture--gate-decomposition-abcdefgsch)
4. [2-Bit (2-Digit) Multiplexed SSD Architecture (`Bit_2_SSD.sch`)](#2-bit-2-digit-multiplexed-ssd-architecture-bit_2_ssdsch)
   * [The Pin Explosion Bottleneck: Direct Drive vs. TDM Scanning](#the-pin-explosion-bottleneck-direct-drive-vs-tdm-scanning)
   * [Persistence of Vision (POV) & Scanning Physics](#persistence-of-vision-pov--scanning-physics)
   * [Detailed Block-by-Block Schematic Breakdown](#detailed-block-by-block-schematic-breakdown)
   * [Time-Division Multiplexing State Sequence & Waveforms](#time-division-multiplexing-state-sequence--waveforms)
5. [End-to-End System Integration: Counter-to-Display Cascade](#end-to-end-system-integration-counter-to-display-cascade)
6. [Synthesizeable Verilog HDL Reference Modules](#synthesizeable-verilog-hdl-reference-modules)
   * [BCD to 7-Segment Decoder (`bcd_to_7seg.v`)](#1-bcd-to-7-segment-decoder-module)
   * [Clock Prescaler / Refresh Generator (`freq_prescaler.v`)](#2-clock-prescaler--refresh-generator)
   * [2-to-1 7-Bit & 8-Bit Multiplexers (`mux_2to1.v`)](#3-2-to-1-bus-multiplexers)
   * [Top-Level 2-Bit Multiplexed Driver (`bit_2_ssd_top.v`)](#4-top-level-2-bit-multiplexed-driver)
7. [FPGA Deployment, UCF Constraints & Pin Mapping](#fpga-deployment-ucf-constraints--pin-mapping)
8. [Hardware Pitfalls, Ghosting Prevention & Troubleshooting](#hardware-pitfalls-ghosting-prevention--troubleshooting)

---

## Project Directory & Module Manifest

| Directory | Primary Module | Circuit Category | Key Architecture / Feature | Module README |
| :--- | :--- | :--- | :--- | :--- |
| **`01_BCD_to_7Segment_Decoder`** | `abcdefg.sch` | Combinational Decoder | 4-bit BCD to 7-segment mapping with 7 modular gate sub-blocks ($a..g$) | [Read Module](file:///c:/Users/himan/OneDrive/Documents/GitHub/Display-Drivers/01_BCD_to_7Segment_Decoder/README.md) |
| **`02_2Bit_Multiplexed_SSD`** | `Bit_2_SSD.sch` | Hybrid Sequential/TDM | Time-Division Multiplexed 2-digit scanning display with clock prescaler | [Read Module](file:///c:/Users/himan/OneDrive/Documents/GitHub/Display-Drivers/02_2Bit_Multiplexed_SSD/README.md) |
| **`assets/`** | Schematics & Waveforms | Visual Artifacts | High-resolution exported schematics and ISim waveform captures | [View Assets](file:///c:/Users/himan/OneDrive/Documents/GitHub/Display-Drivers/assets/README.md) |

---

## Architectural Continuum: From Sequential Registers to Display Drivers

In digital computation, binary data lives inside sequential memory elements and arithmetic counters. However, raw binary vectors ($0000_2 - 1111_2$) are unreadable to human users. The display subsystem provides the critical decoding and physical drive bridge.

```
+---------------------------------------------------------------------------------------------------------+
|                                    DIGITAL SYSTEM DATA CONTINUUM                                        |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|  +---------------------------+       +---------------------------+       +---------------------------+  |
|  | Sequential Architecture   |       | Digital Counters & FSMs   |       | Display Driver Subsystem  |  |
|  | (Flip-Flops & Registers)  | ----> | (MOD-10 BCD Decade Chain) | ----> | (Decoders & Multiplexers) |  |
|  | - SR / MSJK / D / T FFs   |       | - 4-Bit BCD Count (Q3..Q0)|       | - BCD -> 7-Seg (abcdefg)  |  |
|  | - SISO/SIPO/PISO/PIPO     |       | - Automatic 1010_2 Reset  |       | - 2-Bit TDM Scan Driver   |  |
|  +---------------------------+       +---------------------------+       +---------------------------+  |
|               |                                    |                                   |                |
|       [1-Bit Memory Cells]               [Cyclic State Generators]             [Human-Readable Output]  |
+---------------------------------------------------------------------------------------------------------+
```

### The Three Pillars of the Hardware Architecture
1. **Sequential Core ([Sequential-Logic-Architecture](file:///c:/Users/himan/OneDrive/Documents/GitHub/Sequential-Logic-Architecture/README.md))**:
   * **Master-Slave JK (MSJK)** and **D Flip-Flops** eliminate race-around conditions and provide synchronized register stages.
   * **T Flip-Flops** toggle outputs on active clock edges to perform precise $\div 2$ frequency division.
2. **State & Event Counting ([Counters-and-FSms](file:///c:/Users/himan/OneDrive/Documents/GitHub/Counters-and-FSms/README.md))**:
   * **MOD-10 Decade Counters** constrain 4-bit natural binary progressions ($2^4 = 16$) to 10 valid decimal states ($0000_2 - 1001_2$) via feedback truncation ($Q_3 \cdot Q_1 = 1$).
   * These counters output parallel Binary Coded Decimal (BCD) nibbles representing Units ($10^0$), Tens ($10^1$), and Hundreds ($10^2$).
3. **Display Driver Interface ([Display-Drivers](file:///c:/Users/himan/OneDrive/Documents/GitHub/Display-Drivers/README.md))**:
   * **Combinational Decoder (`abcdefg.sch`)**: Converts a 4-bit BCD nibble into 7 individual LED segment controls.
   * **Sequential Prescaler & Multiplexer (`Bit_2_SSD.sch`)**: Leverages high-speed Time-Division Multiplexing to drive multi-digit displays over a unified, shared 7-bit segment bus.

---

## Fundamentals of Seven-Segment Displays (SSDs)

An SSD consists of seven elongated Light Emitting Diodes (LEDs) arranged in a planar figure-8 topology, augmented by an optional 8th circular diode for a decimal point ($DP$).

### Segment Topologies & Naming Conventions

```
                 ================= a =================
               ||                                     ||
               ||                                     ||
               f                                       b
               ||                                     ||
               ||                                     ||
                 ================= g =================
               ||                                     ||
               ||                                     ||
               e                                       c
               ||                                     ||
               ||                                     ||
                 ================= d =================   ( * ) DP
```

* **Standard Segment Labeling**: Labeled clockwise starting from the top horizontal bar: $a \rightarrow b \rightarrow c \rightarrow d \rightarrow e \rightarrow f$, with the middle horizontal bar designated as $g$.
* **Visual Encodings**: Activating distinct subsets of these 7 segments renders all Arabic numerals ($0 - 9$) as well as standard hexadecimal characters ($A, b, C, d, E, F$).

---

### Common Anode (CA) vs. Common Cathode (CC) Configurations

The electrical connection of the internal LED junctions defines how the display must be interfaced to FPGA output pins:

```
        1. COMMON ANODE (CA) CIRCUIT:                     2. COMMON CATHODE (CC) CIRCUIT:
                 +Vcc (+3.3V / +5V)                                      GND (0V)
                         |                                                  |
            +------------+------------+                        +------------+------------+
            |            |            |                        |            |            |
          +---+        +---+        +---+                    +---+        +---+        +---+
          | A |        | A |        | A |                    | K |        | K |        | K |
          |   |        |   |        |   |                    |   |        |   |        |   |
          +---+        +---+        +---+                    +---+        +---+        +---+
            | (LED a)    | (LED b)    | (LED c)                | (LED a)    | (LED b)    | (LED c)
            v            v            v                        ^            ^            ^
          -----        -----        -----                    -----        -----        -----
           \ /          \ /          \ /                      / \          / \          / \
          -----        -----        -----                    -----        -----        -----
            |            |            |                        |            |            |
            |            |            |                        |            |            |
            o a          o b          o c                      o a          o b          o c
      (Drive LOW = 0) (Drive LOW = 0)                   (Drive HIGH = 1) (Drive HIGH = 1)
```

$$\begin{array}{l|c|c|c|c|c}
\mathbf{Topology} & \mathbf{Shared\ Terminal} & \mathbf{Control\ Lines} & \mathbf{Active\ Logic} & \mathbf{FPGA\ Current\ Mode} & \mathbf{Typical\ Board\ Use} \\
\hline
\textbf{Common Anode (CA)} & \text{All Anodes } \rightarrow V_{CC} & \text{Individual Cathodes } (a..g) & \textbf{Active LOW } (\mathbf{0}) & \text{FPGA Sinks Current } (I_{\text{sink}}) & \text{Basys 2, Nexys 4, Nexys A7} \\
\textbf{Common Cathode (CC)} & \text{All Cathodes } \rightarrow \text{GND} & \text{Individual Anodes } (a..g) & \textbf{Active HIGH } (\mathbf{1}) & \text{FPGA Sources Current } (I_{\text{source}}) & \text{Standalone Breadboard SSDs} \\
\end{array}$$

---

### Electrical Characteristics & Resistor Sizing Calculations

Each individual LED segment exhibits a non-linear forward voltage drop ($V_F$) depending on its semiconductor material and emission color:

$$\begin{array}{l|c|c|c}
\mathbf{LED\ Color} & \mathbf{Semiconductor\ Composition} & \mathbf{Typical\ Forward\ Voltage\ } (V_F) & \mathbf{Target\ Operating\ Current\ } (I_F) \\
\hline
\text{Red (Standard)} & \text{GaAsP / GaP} & 1.8\text{ V} - 2.0\text{ V} & 5\text{ mA} - 15\text{ mA} \\
\text{Super Bright Red} & \text{AlInGaP} & 1.9\text{ V} - 2.1\text{ V} & 5\text{ mA} - 10\text{ mA} \\
\text{Green} & \text{GaP / InGaN} & 2.1\text{ V} - 2.4\text{ V} & 10\text{ mA} - 20\text{ mA} \\
\text{Blue / White} & \text{InGaN} & 3.0\text{ V} - 3.3\text{ V} & 10\text{ mA} - 20\text{ mA} \\
\end{array}$$

#### Static Continuous Drive Current Resistor Formula:
$$R_{\text{seg}} = \frac{V_{CCO} - V_F - V_{\text{sat}}}{I_F}$$

Where:
* $V_{CCO}$ = FPGA I/O Bank Voltage ($3.3\text{ V}$ for `LVCMOS33`).
* $V_F$ = LED Forward Voltage drop ($\approx 2.0\text{ V}$ for Red).
* $V_{\text{sat}}$ = Driver saturation voltage (or internal FPGA output buffer drop $\approx 0.1\text{ V} - 0.2\text{ V}$).
* $I_F$ = Desired continuous current per segment ($\approx 10\text{ mA}$).

$$\therefore R_{\text{seg}} = \frac{3.3\text{ V} - 2.0\text{ V} - 0.1\text{ V}}{0.010\text{ A}} = \frac{1.2\text{ V}}{0.010\text{ A}} = 120\ \Omega \quad (\text{Standard Standard Value: } 100\ \Omega - 220\ \Omega)$$

#### Pulsed Current in Multiplexed Mode:
In a time-multiplexed display with $N$ digits and a duty cycle $D = \frac{1}{N}$, each digit is only powered for $\frac{1}{N}$ of the total scanning frame. To maintain an equivalent perceived brightness ($I_{\text{avg}}$), the peak instantaneous pulse current $I_{\text{peak}}$ can be sized up to:
$$I_{\text{peak}} = N \times I_{\text{avg}}$$
*(Subject to the maximum pulsed current ratings in the SSD manufacturer datasheet, typically $< 30\text{ mA} - 50\text{ mA}$ peak).*

---

## Single-Digit BCD-to-7-Segment Decoder (`abcdefg.sch`)

The **BCD-to-7-Segment Decoder** (`abcdefg.sch`) is a combinational logic circuit that accepts a 4-bit binary-coded decimal input $D[3:0] = \{D_3, D_2, D_1, D_0\}$ and synthesizes the 7 boolean functions required to drive segments $a, b, c, d, e, f, g$.

```
                            +-------------------------------------------+
                            |                abcdefg.sch                |
                            |                                           |
                            |   +-------------+                         |
                            |   |  Block [a]  |-----------------------> | a (Segment a)
                            |   +-------------+                         |
                            |   +-------------+                         |
                            |   |  Block [b]  |-----------------------> | b (Segment b)
                            |   +-------------+                         |
                            |   +-------------+                         |
       4-Bit Input Bus      |   |  Block [c]  |-----------------------> | c (Segment c)
   I[3:0] / D[3:0] -------->|   +-------------+                         |
   {D3, D2, D1, D0}         |   +-------------+                         |-----> F[6:0] / SEG[6:0]
                            |   |  Block [d]  |-----------------------> | d (Segment d)
                            |   +-------------+                         |
                            |   +-------------+                         |
                            |   |  Block [e]  |-----------------------> | e (Segment e)
                            |   +-------------+                         |
                            |   +-------------+                         |
                            |   |  Block [f]  |-----------------------> | f (Segment f)
                            |   +-------------+                         |
                            |   +-------------+                         |
                            |   |  Block [g]  |-----------------------> | g (Segment g)
                            |   +-------------+                         |
                            +-------------------------------------------+
```

---

### Functional Specification & Truth Table

$$\begin{array}{c|c|cccc|ccccccc|ccccccc|c}
\mathbf{Digit} & \mathbf{Dec} & \mathbf{D_3} & \mathbf{D_2} & \mathbf{D_1} & \mathbf{D_0} & \mathbf{a_{CC}} & \mathbf{b_{CC}} & \mathbf{c_{CC}} & \mathbf{d_{CC}} & \mathbf{e_{CC}} & \mathbf{f_{CC}} & \mathbf{g_{CC}} & \mathbf{a_{CA}} & \mathbf{b_{CA}} & \mathbf{c_{CA}} & \mathbf{d_{CA}} & \mathbf{e_{CA}} & \mathbf{f_{CA}} & \mathbf{g_{CA}} & \mathbf{Glyph\ Shape} \\
\hline
0 & 0 & 0 & 0 & 0 & 0 & 1 & 1 & 1 & 1 & 1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & \text{Solid Box} \\
1 & 1 & 0 & 0 & 0 & 1 & 0 & 1 & 1 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 1 & 1 & 1 & 1 & \text{Right Bar} \\
2 & 2 & 0 & 0 & 1 & 0 & 1 & 1 & 0 & 1 & 1 & 0 & 1 & 0 & 0 & 1 & 0 & 0 & 1 & 0 & \text{Z-Pattern} \\
3 & 3 & 0 & 0 & 1 & 1 & 1 & 1 & 1 & 1 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 1 & 1 & 0 & \text{Fork Pattern} \\
4 & 4 & 0 & 1 & 0 & 0 & 0 & 1 & 1 & 0 & 0 & 1 & 1 & 1 & 0 & 0 & 1 & 1 & 0 & 0 & \text{Chair Pattern} \\
5 & 5 & 0 & 1 & 0 & 1 & 1 & 0 & 1 & 1 & 0 & 1 & 1 & 0 & 1 & 0 & 0 & 1 & 0 & 0 & \text{S-Pattern} \\
6 & 6 & 0 & 1 & 1 & 0 & 1 & 0 & 1 & 1 & 1 & 1 & 1 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & \text{6 with Top} \\
7 & 7 & 0 & 1 & 1 & 1 & 1 & 1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 1 & 1 & 1 & \text{Top Angle} \\
8 & 8 & 1 & 0 & 0 & 0 & 1 & 1 & 1 & 1 & 1 & 1 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & \text{All 7 ON} \\
9 & 9 & 1 & 0 & 0 & 1 & 1 & 1 & 1 & 1 & 0 & 1 & 1 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & \text{9 with Tail} \\
\hline
\text{Don't Care} & 10..15 & 1 & x & x & x & X & X & X & X & X & X & X & X & X & X & X & X & X & X & \text{Illegal BCD States} \\
\end{array}$$

---

### K-Map Derivations for Segments a through g

By utilizing Karnaugh Maps (K-Maps) across the 4 input variables $D_3, D_2, D_1, D_0$ with minterms $m(0..9)$ and don't care conditions $d(10..15) = X$, we extract the globally minimized Sum-of-Products (SOP) boolean equations.

#### 1. Segment $a$ K-Map Optimization
* **Minterms**: $a = \sum m(0, 2, 3, 5, 6, 7, 8, 9) + \sum d(10, 11, 12, 13, 14, 15)$

$$\begin{array}{c|c|c|c|c}
D_3 D_2 \backslash D_1 D_0 & \mathbf{00} & \mathbf{01} & \mathbf{11} & \mathbf{10} \\
\hline
\mathbf{00} & 1 & 0 & 1 & 1 \\
\mathbf{01} & 0 & 1 & 1 & 1 \\
\mathbf{11} & X & X & X & X \\
\mathbf{10} & 1 & 1 & X & X \\
\end{array}$$

$$\mathbf{a = D_3 + D_1 + D_2 D_0 + \bar{D}_2 \bar{D}_0}$$

---

#### 2. Segment $b$ K-Map Optimization
* **Minterms**: $b = \sum m(0, 1, 2, 3, 4, 7, 8, 9) + \sum d(10, 11, 12, 13, 14, 15)$

$$\begin{array}{c|c|c|c|c}
D_3 D_2 \backslash D_1 D_0 & \mathbf{00} & \mathbf{01} & \mathbf{11} & \mathbf{10} \\
\hline
\mathbf{00} & 1 & 1 & 1 & 1 \\
\mathbf{01} & 1 & 0 & 1 & 0 \\
\mathbf{11} & X & X & X & X \\
\mathbf{10} & 1 & 1 & X & X \\
\end{array}$$

$$\mathbf{b = \bar{D}_2 + \bar{D}_1 \bar{D}_0 + D_1 D_0}$$

---

#### 3. Segment $c$ K-Map Optimization
* **Minterms**: $c = \sum m(0, 1, 3, 4, 5, 6, 7, 8, 9) + \sum d(10, 11, 12, 13, 14, 15)$

$$\begin{array}{c|c|c|c|c}
D_3 D_2 \backslash D_1 D_0 & \mathbf{00} & \mathbf{01} & \mathbf{11} & \mathbf{10} \\
\hline
\mathbf{00} & 1 & 1 & 1 & 0 \\
\mathbf{01} & 1 & 1 & 1 & 1 \\
\mathbf{11} & X & X & X & X \\
\mathbf{10} & 1 & 1 & X & X \\
\end{array}$$

$$\mathbf{c = D_2 + \bar{D}_1 + D_0}$$

---

#### 4. Segment $d$ K-Map Optimization
* **Minterms**: $d = \sum m(0, 2, 3, 5, 6, 8, 9) + \sum d(10, 11, 12, 13, 14, 15)$

$$\mathbf{d = D_3 + \bar{D}_2 \bar{D}_0 + D_1 \bar{D}_0 + D_2 \bar{D}_1 D_0 + \bar{D}_2 D_1}$$

---

#### 5. Segment $e$ K-Map Optimization
* **Minterms**: $e = \sum m(0, 2, 6, 8) + \sum d(10, 11, 12, 13, 14, 15)$

$$\mathbf{e = \bar{D}_2 \bar{D}_0 + D_1 \bar{D}_0}$$

---

#### 6. Segment $f$ K-Map Optimization
* **Minterms**: $f = \sum m(0, 4, 5, 6, 8, 9) + \sum d(10, 11, 12, 13, 14, 15)$

$$\mathbf{f = D_3 + D_2 \bar{D}_1 + D_2 \bar{D}_0 + \bar{D}_1 \bar{D}_0}$$

---

#### 7. Segment $g$ K-Map Optimization
* **Minterms**: $g = \sum m(2, 3, 4, 5, 6, 8, 9) + \sum d(10, 11, 12, 13, 14, 15)$

$$\mathbf{g = D_3 + D_2 \bar{D}_1 + \bar{D}_2 D_1 + D_1 \bar{D}_0}$$

---

### Schematic Architecture & Gate Decomposition (`abcdefg.sch`)

In the ISE Project Navigator schematic (`abcdefg.sch`):
1. **Input Bus Conditioning**: The 4-bit bus `I[3:0]` enters the top-level boundary. Bus rippers extract individual nets $D_3, D_2, D_1, D_0$ along with their inverted complements $\bar{D}_3, \bar{D}_2, \bar{D}_1, \bar{D}_0$.
2. **Modular Block Decomposition**: The circuit is split into 7 separate functional blocks labeled `a`, `b`, `c`, `d`, `e`, `f`, `g`. Each sub-block contains the dedicated AND-OR gate array corresponding to its respective minimized boolean equation.
3. **Output Bus Aggregation**: The individual outputs $a$ through $g$ converge into the unified 7-bit output bus `F[6:0]` (where `F[0]=a, F[1]=b, ..., F[6]=g`), ready for modular instantiation in higher-level systems.

#### Gate-Level Schematic Capture (`abcdefg.sch`)

<img width="1920" height="1005" alt="abcdefg_sch" src="https://github.com/user-attachments/assets/350ef433-311d-4acb-8327-6d7ba656ebfa" />

---

## 2-Bit (2-Digit) Multiplexed SSD Architecture (`Bit_2_SSD.sch`)

### The Pin Explosion Bottleneck: Direct Drive vs. TDM Scanning

Directly connecting every display segment to an FPGA output pin works for 1 digit, but quickly becomes unfeasible as digit count grows:

$$\begin{array}{l|c|c|c|c|c}
\mathbf{Display\ Size} & \mathbf{Direct\ Drive\ Pins} & \mathbf{TDM\ Multiplexed\ Pins} & \mathbf{Pin\ Savings\ (\%)} & \mathbf{FPGA\ Routing\ Congestion} & \mathbf{Board\ Complexity} \\
\hline
\text{1 Digit (0-9)} & 7\text{ pins} + 1\text{ AN} = 8 & 7\text{ segs} + 1\text{ AN} = 8 & 0\% & \text{Minimal} & \text{Low} \\
\textbf{2 Digits (00-99)} & 14\text{ pins} + 2\text{ AN} = \mathbf{16} & 7\text{ segs} + 2\text{ AN} = \mathbf{9} & \mathbf{43.7\%} & \text{Low} & \text{Low} \\
\text{4 Digits (0000-9999)} & 28\text{ pins} + 4\text{ AN} = 32 & 7\text{ segs} + 4\text{ AN} = 11 & \mathbf{65.6\%} & \text{Moderate} & \text{Moderate} \\
\text{8 Digits (Multi-Word)} & 56\text{ pins} + 8\text{ AN} = 64 & 7\text{ segs} + 8\text{ AN} = 15 & \mathbf{76.5\%} & \text{High} & \text{High} \\
\end{array}$$

**Conclusion**: Time-Division Multiplexing scales linearly with formula $N_{\text{pins}} = 7 + N_{\text{digits}}$, allowing an FPGA to control complex multi-digit displays while consuming minimal I/O pins.

---

### Persistence of Vision (POV) & Scanning Physics

**Persistence of Vision (POV)** is the human visual phenomenon where the retina and visual cortex retain an illuminated image for approximately $\Delta t_{\text{eye}} \approx 20\text{ ms} - 40\text{ ms}$ after the optical stimulus is removed.

```
       +-------------------------------------------------------------------------------+
       |                         POV INTEGRATION OVER TIME                             |
       +-------------------------------------------------------------------------------+
       |                                                                               |
       |  Digit 0:  [ ON (1ms) ] [ OFF (1ms)] [ ON (1ms) ] [ OFF (1ms)] ...            |
       |  Digit 1:  [ OFF (1ms)] [ ON (1ms) ] [ OFF (1ms)] [ ON (1ms) ] ...            |
       |                                                                               |
       |  Human Eye Perception:                                                        |
       |  Digit 0:  ====================== CONTINUOUSLY ON ======================      |
       |  Digit 1:  ====================== CONTINUOUSLY ON ======================      |
       +-------------------------------------------------------------------------------+
```

#### Refresh Rate Frequency Design Bounds:
1. **Lower Bound ($f_{\text{scan}} > 60\text{ Hz}$)**: If the scanning frequency drops below $50\text{ Hz} - 60\text{ Hz}$ ($T_{\text{frame}} > 16.6\text{ ms}$), human perception detects **flicker**, causing eye strain.
2. **Upper Bound ($f_{\text{scan}} < 50\text{ kHz}$)**: If the scanning frequency is driven too high ($> 50\text{ kHz}$), parasitic capacitance in the PCB traces and junction capacitance of the display LEDs cause **ghosting** (bleeding of the previous digit's pattern onto the next digit during transition).
3. **Optimal Industrial Target**:
   $$f_{\text{refresh\_per\_digit}} \approx 100\text{ Hz} - 1\text{ kHz} \quad (T_{\text{slot}} \approx 0.5\text{ ms} - 5\text{ ms})$$

---

### Detailed Block-by-Block Schematic Breakdown

The top-level schematic `Bit_2_SSD.sch` integrates 4 key functional blocks:

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

#### 1. Clock Prescaler (`Frequency_100KHz`)
* **Role**: Accepts the master crystal oscillator clock (`CLK`, $50\text{ MHz}$ or $100\text{ MHz}$) and divides it down via a chain of synchronous/asynchronous flip-flop counters.
* **Output Net (`Q`)**: A clean 50% duty cycle square wave operating at the target multiplexing frequency ($f_{\text{scan}} \approx 100\text{ Hz} - 1\text{ kHz}$).
* **Interface**: Controlled by global asynchronous clear (`CLR`) to initialize in a known state.

#### 2. Dual BCD-to-7-Segment Decoder Instances (`abcdefg`)
* **Upper Instance (`XLXI_1`)**: Converts the 4-bit BCD input `IN1[3:0]` (Digit 1 / Tens digit) into a dedicated 7-bit segment pattern.
* **Lower Instance (`XLXI_2`)**: Converts the 4-bit BCD input `IN0[3:0]` (Digit 0 / Units digit) into a dedicated 7-bit segment pattern.
* **Parallel Operation**: Both decoders operate combinationally in parallel with near-zero latency, ensuring valid segment data is continuously presented at the multiplexer inputs.

#### 3. 7-Bit 2-to-1 Segment Multiplexer (`Mux_2_1_7bit`)
* **Role**: Acts as a high-speed data bus selector.
* **Operation**:
  $$\text{SEG}[6:0] = \begin{cases} \text{Decoded}(IN0[3:0]), & \text{if } Q = 0 \\ \text{Decoded}(IN1[3:0]), & \text{if } Q = 1 \end{cases}$$
* **Hardware Efficiency**: Connects all 7 segment pins of both digits together on the PCB, using this single multiplexer to steer data.

#### 4. 8-Bit / 2-Bit Anode Multiplexer (`Mux_2_1_8bit`)
* **Role**: Coordinates the power delivery to the common anodes/cathodes.
* **Inputs Tied to Rails**: Hardwired `VCC` (Logic 1) and `GND` (Logic 0) nets establish predefined active-low digit enable words:
  * For $Q = 0 \rightarrow \text{Output } AN[7:0] = 8\text{'b}1111\_1110$ (Only Digit 0 is grounded/powered).
  * For $Q = 1 \rightarrow \text{Output } AN[7:0] = 8\text{'b}1111\_1101$ (Only Digit 1 is grounded/powered).
* **Zero Phase Error**: Controlled by the identical select line $Q$, guaranteeing exact synchronization between segment data and anode power.

---

### Time-Division Multiplexing State Sequence & Waveforms

$$\begin{array}{c|c|c|c|c|c}
\mathbf{Clock\ Phase\ (Q)} & \mathbf{Active\ Time\ Slot} & \mathbf{Mux\_2\_1\_7bit\ Output} & \mathbf{Mux\_2\_1\_8bit\ Output\ (AN)} & \mathbf{Digit\ 0\ Status} & \mathbf{Digit\ 1\ Status} \\
\hline
\mathbf{Q = 0} & \text{Slot 0 } (0 - 1.0\text{ ms}) & \text{Segments for } IN0 & \mathbf{AN0 = 0\ (ON)}, AN1 = 1\text{ (OFF)} & \mathbf{ILLUMINATED} & \text{DARK} \\
\mathbf{Q = 1} & \text{Slot 1 } (1.0 - 2.0\text{ ms}) & \text{Segments for } IN1 & AN0 = 1\text{ (OFF)}, \mathbf{AN1 = 0\ (ON)} & \text{DARK} & \mathbf{ILLUMINATED} \\
\end{array}$$

```
TIMING DIAGRAM & BUS MULTIPLEXING SEQUENCE:

               +---------------+               +---------------+
CLK_SCAN (Q)   |   Slot 0 (D0) |   Slot 1 (D1) |   Slot 0 (D0) |   Slot 1 (D1)
             --+               +---------------+               +---------------

AN0 (D0 EN)  __________________-----------------________________---------------- (Active LOW)

AN1 (D1 EN)  ------------------________________-----------------________________ (Active LOW)

SEG[6:0]     ====== D0 Segments ===== D1 Segments ===== D0 Segments ===== D1 Segments ===

Visible:     [ Display Digit 0 ]               [ Display Digit 0 ]
                               [ Display Digit 1 ]               [ Display Digit 1 ]
             ================== TOTAL PERCEIVED STEADY DISPLAY ===================
```

#### Top-Level Schematic Capture (`Bit_2_SSD.sch`)

<img width="1920" height="1005" alt="bit_2_ssd_sch" src="https://github.com/user-attachments/assets/3e242c3d-bafc-44c1-9196-25023e0c4d85" />

---

## End-to-End System Integration: Counter-to-Display Cascade

The full power of this modular architecture becomes evident when cascading the sequential counters and display drivers together into a complete 2-Digit Decimal Counter System ($00$ to $99$):

```
+---------------------------------------------------------------------------------------------------------+
|                                COMPLETE 2-DIGIT DECIMAL STOPWATCH SYSTEM                                |
+---------------------------------------------------------------------------------------------------------+
|                                                                                                         |
|  Master Clock (100MHz)                                                                                  |
|        |                                                                                                |
|        v                                                                                                |
|  +--------------------+                                                                                 |
|  | 1Hz Clock Divider  |                                                                                 |
|  +--------------------+                                                                                 |
|        | 1Hz Pulse                                                                                      |
|        v                                                                                                |
|  +--------------------+       Q_Units[3:0]       +--------------------+                                 |
|  | MOD-10 BCD Counter |------------------------->| IN0                |                                 |
|  | (Units Digit: 0-9) |                          |                    |                                 |
|  +--------------------+                          |                    |                                 |
|        | Rollover Carry (TC at 9)                |                    |                                 |
|        v                                         | 2-Bit Multiplexed  |                                 |
|  +--------------------+       Q_Tens[3:0]        | Display Driver     |==== SEG[6:0] ===> [ 7-Seg LEDs ]|
|  | MOD-10 BCD Counter |------------------------->| IN1 (Bit_2_SSD.sch)|                                 |
|  | (Tens Digit: 0-9)  |                          |                    |==== AN[1:0]  ===> [ Dig Anodes ]|
|  +--------------------+                          |                    |                                 |
|                                                  |                    |                                 |
|  Scan Clock (500Hz) ---------------------------->| CLK_SCAN           |                                 |
|                                                  +--------------------+                                 |
+---------------------------------------------------------------------------------------------------------+
```

---

## Synthesizeable Verilog HDL Reference Modules

For teams utilizing Verilog HDL synthesis alongside schematic captures in Xilinx ISE, the complete synthesizeable code suite is detailed below.

### 1. BCD-to-7-Segment Decoder Module
```verilog
// File: bcd_to_7seg.v
// Active-Low Common Anode Output (invert for Common Cathode)
`timescale 1ns / 1ps

module bcd_to_7seg (
    input  wire [3:0] bcd,       // 4-bit BCD input (0-9)
    output reg  [6:0] seg        // 7-bit segment bus: {g, f, e, d, c, b, a}
);

    always @(*) begin
        case (bcd)
            4'h0: seg = 7'b1000000; // Display 0 (Active LOW: a,b,c,d,e,f on)
            4'h1: seg = 7'b1111001; // Display 1 (b,c on)
            4'h2: seg = 7'b0100100; // Display 2 (a,b,d,e,g on)
            4'h3: seg = 7'b0110000; // Display 3 (a,b,c,d,g on)
            4'h4: seg = 7'b0011001; // Display 4 (b,c,f,g on)
            4'h5: seg = 7'b0010010; // Display 5 (a,c,d,f,g on)
            4'h6: seg = 7'b0000010; // Display 6 (a,c,d,e,f,g on)
            4'h7: seg = 7'b1111000; // Display 7 (a,b,c on)
            4'h8: seg = 7'b0000000; // Display 8 (all on)
            4'h9: seg = 7'b0010000; // Display 9 (a,b,c,d,f,g on)
            default: seg = 7'b1111111; // Blank display on invalid input
        endcase
    end

endmodule
```

### 2. Clock Prescaler / Refresh Generator
```verilog
// File: freq_prescaler.v
`timescale 1ns / 1ps

module freq_prescaler #(
    parameter INPUT_FREQ_HZ  = 100_000_000, // 100MHz Master FPGA Clock
    parameter REFRESH_FREQ_HZ = 500         // 500Hz Scanning Refresh Clock
)(
    input  wire clk,
    input  wire rst,
    output reg  clk_out
);

    localparam integer DIVISOR = INPUT_FREQ_HZ / (2 * REFRESH_FREQ_HZ);
    reg [31:0] counter;

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            counter <= 32'd0;
            clk_out <= 1'b0;
        end else begin
            if (counter >= DIVISOR - 1) begin
                counter <= 32'd0;
                clk_out <= ~clk_out; // Toggle refresh clock
            end else begin
                counter <= counter + 1'b1;
            end
        end
    end

endmodule
```

### 3. 2-to-1 Bus Multiplexers
```verilog
// File: mux_2to1_7bit.v
`timescale 1ns / 1ps

module mux_2to1_7bit (
    input  wire [6:0] in0,
    input  wire [6:0] in1,
    input  wire       sel,
    output wire [6:0] out
);
    assign out = (sel == 1'b0) ? in0 : in1;
endmodule

// File: mux_2to1_8bit.v
module mux_2to1_8bit (
    input  wire [7:0] in0,
    input  wire [7:0] in1,
    input  wire       sel,
    output wire [7:0] out
);
    assign out = (sel == 1'b0) ? in0 : in1;
endmodule
```

### 4. Top-Level 2-Bit Multiplexed Driver
```verilog
// File: bit_2_ssd_top.v
`timescale 1ns / 1ps

module bit_2_ssd_top (
    input  wire        clk,       // 100MHz FPGA Master Clock
    input  wire        clr,       // Asynchronous Active-High Reset
    input  wire [3:0]  in0,       // Digit 0 (Units) BCD Data
    input  wire [3:0]  in1,       // Digit 1 (Tens) BCD Data
    output wire [6:0]  seg,       // Common Segment Bus {g,f,e,d,c,b,a}
    output wire [7:0]  an         // Anode Enable Bus
);

    wire        scan_clk;
    wire [6:0]  seg_digit0;
    wire [6:0]  seg_digit1;

    // 1. Clock Prescaler Submodule
    freq_prescaler #(
        .INPUT_FREQ_HZ(100_000_000),
        .REFRESH_FREQ_HZ(500)
    ) u_prescaler (
        .clk(clk),
        .rst(clr),
        .clk_out(scan_clk)
    );

    // 2. Parallel BCD-to-7-Segment Decoders
    bcd_to_7seg u_dec0 (.bcd(in0), .seg(seg_digit0));
    bcd_to_7seg u_dec1 (.bcd(in1), .seg(seg_digit1));

    // 3. Segment Bus 2:1 Multiplexer
    mux_2to1_7bit u_seg_mux (
        .in0(seg_digit0),
        .in1(seg_digit1),
        .sel(scan_clk),
        .out(seg)
    );

    // 4. Anode Enable 2:1 Multiplexer
    // in0 = 8'b1111_1110 (Digit 0 active low)
    // in1 = 8'b1111_1101 (Digit 1 active low)
    mux_2to1_8bit u_an_mux (
        .in0(8'b1111_1110),
        .in1(8'b1111_1101),
        .sel(scan_clk),
        .out(an)
    );

endmodule
```

---

## FPGA Deployment, UCF Constraints & Pin Mapping

To deploy the design onto physical hardware, pin mappings are defined in the User Constraints File (`.ucf`) for Xilinx ISE.

### Target: Digilent Nexys 4 / Nexys A7 (Xilinx Artix-7 `xc7a100t-csg324`)

```ucf
# Master 100MHz Crystal Oscillator
NET "clk"       LOC = "E3"  | IOSTANDARD = "LVCMOS33";
NET "clk" TNM_NET = "sys_clk_pin";
TIMESPEC "TS_sys_clk_pin" = PERIOD "sys_clk_pin" 100 MHz HIGH 50%;

# Global Asynchronous Reset (Center Pushbutton)
NET "clr"       LOC = "N17" | IOSTANDARD = "LVCMOS33";

# 4-Bit BCD Input 0 (Switches SW3..SW0 - Digit 0 Units)
NET "in0<0>"    LOC = "J15" | IOSTANDARD = "LVCMOS33";
NET "in0<1>"    LOC = "L16" | IOSTANDARD = "LVCMOS33";
NET "in0<2>"    LOC = "M13" | IOSTANDARD = "LVCMOS33";
NET "in0<3>"    LOC = "R15" | IOSTANDARD = "LVCMOS33";

# 4-Bit BCD Input 1 (Switches SW7..SW4 - Digit 1 Tens)
NET "in1<0>"    LOC = "R17" | IOSTANDARD = "LVCMOS33";
NET "in1<1>"    LOC = "T18" | IOSTANDARD = "LVCMOS33";
NET "in1<2>"    LOC = "U18" | IOSTANDARD = "LVCMOS33";
NET "in1<3>"    LOC = "R13" | IOSTANDARD = "LVCMOS33";

# 7-Segment Cathode Lines (Active-Low: seg[0]=a, seg[1]=b, ..., seg[6]=g)
NET "seg<0>"    LOC = "T10" | IOSTANDARD = "LVCMOS33"; # Segment a
NET "seg<1>"    LOC = "R10" | IOSTANDARD = "LVCMOS33"; # Segment b
NET "seg<2>"    LOC = "K16" | IOSTANDARD = "LVCMOS33"; # Segment c
NET "seg<3>"    LOC = "K13" | IOSTANDARD = "LVCMOS33"; # Segment d
NET "seg<4>"    LOC = "P15" | IOSTANDARD = "LVCMOS33"; # Segment e
NET "seg<5>"    LOC = "T11" | IOSTANDARD = "LVCMOS33"; # Segment f
NET "seg<6>"    LOC = "L18" | IOSTANDARD = "LVCMOS33"; # Segment g

# Common Anode Digit Enables (Active-Low)
NET "an<0>"     LOC = "J17" | IOSTANDARD = "LVCMOS33"; # Digit 0 Anode
NET "an<1>"     LOC = "J18" | IOSTANDARD = "LVCMOS33"; # Digit 1 Anode
NET "an<2>"     LOC = "T9"  | IOSTANDARD = "LVCMOS33"; # Digit 2 (Disabled)
NET "an<3>"     LOC = "J14" | IOSTANDARD = "LVCMOS33"; # Digit 3 (Disabled)
NET "an<4>"     LOC = "P14" | IOSTANDARD = "LVCMOS33"; # Digit 4 (Disabled)
NET "an<5>"     LOC = "T14" | IOSTANDARD = "LVCMOS33"; # Digit 5 (Disabled)
NET "an<6>"     LOC = "K2"  | IOSTANDARD = "LVCMOS33"; # Digit 6 (Disabled)
NET "an<7>"     LOC = "U13" | IOSTANDARD = "LVCMOS33"; # Digit 7 (Disabled)
```

---

## Hardware Pitfalls, Ghosting Prevention & Troubleshooting

```
+---------------------------------------------------------------------------------------------------------+
|                                    TROUBLESHOOTING & DESIGN MATRIX                                      |
+---------------------------------------------------------------------------------------------------------+
| Symptom                 | Root Cause                               | Engineering Solution               |
+-------------------------+------------------------------------------+------------------------------------+
| Ghosting / Shadowing    | Anode switches ON before previous        | Insert Blanking / Dead-Time during |
| between adjacent digits | segment data clears (parasitic C charge) | anode transition (Turn off anodes) |
+-------------------------+------------------------------------------+------------------------------------+
| Visible Display Flicker | Scanning clock frequency < 60Hz          | Increase prescaler output frequency|
|                         | (T_frame > 16.6ms)                       | to target 200Hz - 1kHz             |
+-------------------------+------------------------------------------+------------------------------------+
| Dim / Uneven Segments   | Missing current limiting resistors or    | Place 100-220 Ohm series resistors |
|                         | FPGA sourcing current beyond pin limits  | on segment lines; use BJT switches |
+-------------------------+------------------------------------------+------------------------------------+
| Inverted Numerical Display| Polarity mismatch (CA table on CC board)| Invert decoder output bits in RTL  |
| (All segments inverted) | or vice versa                            | (`seg = ~seg`)                     |
+-------------------------+------------------------------------------+------------------------------------+
```

> [!IMPORTANT]
> **Ghosting Prevention Best Practice**: When transitioning between digits, momentarily assert `an = 8'b1111_1111` (all anodes disabled) for $1\text{ }\mu\text{s}$ while the segment multiplexer stabilizes. This completely eliminates faint residual numeral shadowing caused by PCB capacitance.

---

<div align="center">

**Developed with Xilinx ISE Design Suite • Targeting Spartan & Artix FPGA Architectures**

</div>
