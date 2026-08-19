# Display Drivers Visual Artifacts & Schematics

This directory contains visual captures, gate-level schematics, and simulation waveforms exported from **Xilinx ISE Design Suite**.

---

### 1. BCD-to-7-Segment Decoder (`abcdefg.sch`)
**Description**: Gate-level combinational schematic mapping 4-bit BCD input $D[3:0]$ to segments $a, b, c, d, e, f, g$.

<img width="1920" height="1005" alt="abcdefg_sch" src="https://github.com/user-attachments/assets/350ef433-311d-4acb-8327-6d7ba656ebfa" />

---

### 2. 2-Bit Time-Division Multiplexed Display Driver (`Bit_2_SSD.sch`)
**Description**: Top-level system schematic integrating clock prescaler (`Frequency_100KHz`), dual `abcdefg` decoders, `Mux_2_1_7bit`, and `Mux_2_1_8bit` anode selector.

<img width="1920" height="1005" alt="bit_2_ssd_sch" src="https://github.com/user-attachments/assets/3e242c3d-bafc-44c1-9196-25023e0c4d85" />
