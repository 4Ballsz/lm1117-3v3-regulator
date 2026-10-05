# Output verification

DRC confirms the board matches its rules inside Altium. These checks were run on the **exported fabrication files** and the PCB database, which is what a fab actually receives.

## Design rule check

- **0 violations, 0 warnings** in Altium DRC ([`DRC_report.html`](DRC_report.html)).

## Gerber and drill format

| Check | Result |
|---|---|
| Gerber format | RS274X (extended), apertures embedded (`%ADD…%`) |
| Units / precision | millimetres (`%MOMM*%`), 4:4 (`%FSLAX44Y44*%`), absolute |
| Layers | GTL, GBL, GTO, GTS, GBS, GM (board profile), GM1 (mechanical 1) |
| Drill file | Excellon, `METRIC`, `FILE_FORMAT=4:4`, plated |
| Board outline | 28.00 × 17.00 mm |

## Drill-to-pad alignment

Every drill hit was matched to the nearest pad flash on both copper layers:

| Hole | Diameter | Offset from pad centre (top / bottom) |
|---|---|---|
| J1 pin 1 | 1.10 mm | 0.0 / 0.0 µm |
| J1 pin 2 | 1.10 mm | 0.0 / 0.0 µm |
| J2 pin 1 | 1.10 mm | 0.0 / 0.0 µm |
| J2 pin 2 | 1.10 mm | 0.0 / 0.0 µm |
| GND via (left) | 0.71 mm | 0.0 / 0.0 µm |
| GND via (right) | 0.71 mm | 0.0 / 0.0 µm |

The Gerbers and drill file share the same origin.

## Netlist (from the PCB)

| Net | Pads |
|---|---|
| 5V | J1.1, U1.3 (V_IN), C1.+ |
| 3V3 | U1.2 (V_OUT), U1.4 (tab), C2.+, R1.2, R3.2, J2.1 |
| ADJ | U1.1, R1.1, R2.2 |
| LED | R3.1, D1.1 (anode) |
| GND | J1.2, J2.2, C1.−, C2.−, R2.1, D1.2 (cathode), 2 vias |

Matches the schematic pin for pin. R1 sits between OUT and ADJ, and R2 between ADJ and GND.

## Pours and copper

- **Top pour on 3V3, bottom pour on GND.** The opposite would short the output, since the tab is V_OUT.
- **3V3 copper connected to the U1 tab: 382 mm²**, including 3V3 pads and traces. That agrees with Altium's 346 mm² for the polygon alone.
- **No floating copper.** The top layer has 7 copper islands, and each belongs to a net with pads.
- **All three top-side GND islands reach the bottom plane:** one through J1.2, one through J2.2 plus a via, one through the left via.

## Polarity

- **C1, C2:** + pad on the positive rail; silk dot on the + side.
- **D1:** symbol is anode→cathode on pins 1→2; pin 2 is on GND and carries the silkscreen cathode bar.
- **J1, J2:** pin 1 (the rail) is the square pad, marked with a silk dot.

## Library links

Each resistor links to its own workspace component, so Altium's BOM keeps them apart:

| Ref | Component | MPN |
|---|---|---|
| R1 | CMP-009-00178-1 | RK73H2ATTD2000F (200 Ω) |
| R2 | CMP-009-00177-1 | RK73H2ATTD3240F (324 Ω) |
| R3 | CMP-009-00179-1 | RK73H2ATTD4320F (432 Ω) |

R2 originally shared R1's component, so Altium's BOM listed it as 200 Ω. I gave it its own component and pushed the change to the PCB with an ECO. That was a link-only change: every pad, track, via and pour is identical before and after, so the Gerbers didn't need regenerating. DRC re-run afterwards: 0 violations.

## Known discrepancies

- `PCB2.GM1` (Mechanical 1) contains a "U1" assembly label as well as the outline. Use `PCB2.GM` as the outline layer.
