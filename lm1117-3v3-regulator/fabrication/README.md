# Fabrication

`LM1117_Regulator_Rev1_gerbers.zip` contains everything a fab needs. The same files are unzipped in [`gerbers/`](gerbers/).

| File | Layer |
|---|---|
| `PCB2.GTL` | Top copper (3V3 pour) |
| `PCB2.GBL` | Bottom copper (GND pour) |
| `PCB2.GTO` | Top silkscreen |
| `PCB2.GTS` | Top solder mask |
| `PCB2.GBS` | Bottom solder mask |
| `PCB2.GM` | **Board outline — use this one** |
| `PCB2.GM1` | Mechanical 1 (outline plus a "U1" assembly label) |
| `PCB2.TXT` | NC drill, Excellon, plated |

Format: RS274X, millimetres, 4:4, absolute. Drill: metric, 4:4. `PCB2_drill_report.txt` lists the tools.

## Ordering

| Parameter | Value |
|---|---|
| Layers | 2 |
| Size | 28 × 17 mm |
| Thickness | **1.6 mm** |
| Copper | 1 oz |
| Min trace / space | 0.254 / 0.254 mm |
| Min hole | 0.71 mm |
| Surface finish | HASL or ENIG |
| Solder mask | Any colour |

- **Thickness:** the Altium stackup is the 0.41 mm default. Order 1.6 mm so the headers have a rigid board to plug into. Nothing in the Gerbers depends on thickness.
- **Outline layer:** `PCB2.GM1` also carries a small "U1" label inside the board. Some fabs read Mechanical 1 as the routing path. Point them at `PCB2.GM`, or delete `PCB2.GM1` from the zip if their uploader complains.
