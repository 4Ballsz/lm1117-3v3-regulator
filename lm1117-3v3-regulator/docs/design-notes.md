# Design notes

Full working behind every number in the README. Datasheet references are to the TI LM1117 datasheet (SNOS412) unless stated otherwise.

---

## Datasheet values used

| What | Value | Where |
|---|---|---|
| Tab connection | **V_OUT**, not ground | LM1117 Fig. 6-1 |
| SOT-223 pins | 1 = ADJ, 2 = V_OUT, 3 = V_IN, tab = V_OUT | LM1117 Table 6-1 |
| Reference voltage | 1.238 / 1.25 / 1.262 V at 25 °C (±0.96 %) | LM1117 §7.5 |
| | 1.225 – 1.270 V over 0–125 °C, 10–800 mA | |
| ADJ pin current | 60 µA typ, 120 µA max | LM1117 §7.5 |
| Minimum load current | 1.7 mA typ, 5 mA max | LM1117 §7.5 |
| Dropout at 500 mA | 1.15 V typ, 1.25 V max over 0–125 °C | LM1117 §7.5 |
| Output capacitor | ≥ 10 µF, ESR **0.3 – 22 Ω** | LM1117 §9.2.2.1.3 |
| Junction temperature | 125 °C operating, 150 °C absolute max | LM1117 §7.3, §7.1 |
| θJA, SOT-223 | 136 °C/W bare pad → 66 °C/W at 1 in² | LM1117 Table 9-2 |
| LED forward voltage | 1.5 – 2.4 V at 20 mA, no typical given | LTST-C170KRKT datasheet |
| Capacitor ESR | 6 Ω | F921A106MPA datasheet |

---

## Divider: R1 = 200 Ω, R2 = 324 Ω

The LM1117 holds Vref = 1.25 V between OUT and ADJ, so the resistor ratio sets the output.

    Vout = Vref × (1 + R2/R1)

Rearranging for the ratio:

    R2/R1 = Vout/Vref − 1
          = 3.3/1.25 − 1
          = 1.64

Picking R1 = 200 Ω:

    R2   = 1.64 × 200        = 328 Ω   →  E96: 324 Ω
    Vout = 1.25 × (1 + 324/200) = 3.275 V
    Idiv = Vref/R1 = 1.25/200   = 6.25 mA

**ADJ pin current.** The simple equation assumes the same current flows through both resistors. The ADJ pin also pushes its own current into R2:

    Vout = Vref × (1 + R2/R1) + Iadj × R2

    60 µA  × 324 Ω = 19 mV   →  3.294 V   (−0.2 %)
    120 µA × 324 Ω = 39 mV   →  3.314 V   (+0.4 %)

**Why the values are low.**

- First attempt was 1 kΩ / 1.65 kΩ. The ADJ term was then 99–198 mV (+3 % to +6 %), worse than the resistor tolerance.
- Dividing both by 5 brought it to 19–39 mV.
- The divider current (6.25 mA) is now ~100× the ADJ current, and only 1.25 % of the 500 mA load.
- 6.25 mA also exceeds the 5 mA worst-case minimum load. With the LED's 3 mA, the regulator sees 9.3 mA even with J2 unplugged, so it stays in regulation with no load.
- 324 Ω beat 332 Ω because its rounding error goes negative and partly cancels the ADJ error.

**Power check:**

    P = I² × R
    R1: (6.25 mA)² × 200 Ω = 7.8 mW    } vs 250 mW rating
    R2: (6.25 mA)² × 324 Ω = 12.7 mW   }

---

## Tolerance

Resistor extremes come from pushing the two resistors in opposite directions:

    worst high:  R1 −1 %, R2 +1 %  →  198 Ω / 327.24 Ω  →  ratio +1.25 %
    worst low:   R1 +1 %, R2 −1 %  →  202 Ω / 320.76 Ω  →  ratio −1.22 %

So ±1 % resistors give about ±1.2 % on the output.

Stacking the reference and ADJ current at their limits as well (low end uses typical Iadj, since the datasheet gives no minimum):

| Conditions | Output range | vs 3.3 V |
|---|---|---|
| 25 °C | 3.22 – 3.39 V | −2.3 % / +2.6 % |
| 0–125 °C, 10–800 mA | 3.19 – 3.41 V | −3.4 % / +3.3 % |

A 3.3 V rail is normally ±5 %, so 3.135 – 3.465 V. **Both cases fit.** Even assuming zero ADJ current, the low end only drops to 3.17 V.

- 1 % resistors are enough; 0.1 % would buy little.
- Over the full range the reference tolerance dominates, not the resistors.
- This is a straight worst-case stack. An RSS estimate would be tighter.

---

## Thermal

A linear regulator burns the input–output difference as heat, and the same current flows in and out:

    P = (Vin − Vout) × Iload
      = (5 − 3.3) × 0.5
      = 0.85 W

Cross-check: 2.5 W in, 1.65 W out, 0.85 W left as heat.

### Efficiency

    η = Pout/Pin = (Vout × I)/(Vin × I) = Vout/Vin
      = 3.3/5 = 66 %

- The current cancels, so efficiency depends only on the voltage ratio, not on the load.
- Layout can't improve it. The only levers are a smaller input–output gap or a switching regulator.

### Junction temperature

    Tj = Ta + P × θJA

θJA depends on how much copper the tab is soldered to. From Table 9-2, SOT-223, top-side copper only:

| Copper area | θJA (°C/W) | Rise at 0.85 W | Tj at 25 °C | Tj at 45 °C |
|---|---|---|---|---|
| 0.0123 in² (bare pad) | 136 | 115.6 °C | 140.6 °C | 160.6 °C |
| 0.066 in² | 123 | 104.6 °C | 129.6 °C | 149.6 °C |
| 0.3 in² (planned) | 84 | 71.4 °C | 96.4 °C | 116.4 °C |
| **0.53 in² (as built)** | **75** | **63.8 °C** | **88.8 °C** | **108.8 °C** |

- Against the 125 °C operating limit, the bare pad and 0.066 in² both fail. The pour isn't optional.
- I planned for 0.3 in² (≈194 mm²). The pour, filling the space around the parts, came out at **346 mm² (0.54 in²)**, so the 0.53 in² row applies: 36 °C of margin at 25 °C and 16 °C at 45 °C.
- Returns flatten out: 0.0123 → 0.3 in² buys 52 °C/W, while 0.3 → 1.0 in² buys only 18 °C/W more.
- TI's numbers assume still air on their test board. A real enclosure will run hotter.

### Dropout

    headroom = Vin − Vout = 5 − 3.3 = 1.7 V
    worst-case dropout at 500 mA = 1.25 V
    margin = 0.45 V   →  regulates down to Vin ≈ 4.55 V

---

## Capacitors: both F921A106MPA

- The LM1117 uses the output capacitor's ESR to damp its feedback loop. The 0.3–22 Ω window is a stability requirement, not a parasitic to minimise.
- A ceramic sits around 5 mΩ, below the floor, and could let the loop oscillate. That can still read about 3.3 V on a multimeter.
- Tantalum-polymer parts fail the same test, so the number to check is ESR, not the word "tantalum".
- Picked 10 µF, 10 V, ESR 6 Ω, mid-window.

Voltage rating:

    rating ≥ 2 × operating voltage
    C1: 5.0 V × 2 = 10 V
    C2: 3.3 V × 2 = 6.6 V  →  10 V

2× because tantalums fail short when over-voltaged. One 10 V part covers both positions.

- The caps are ±20 %, so worst case is 8 µF against the 10 µF guideline. No ±10 % part exists in 0805 at 10 V. More capacitance only helps stability, so I accepted it.
- The optional ADJ bypass cap is left off. It only improves ripple rejection.

---

## LED: D1 + R3 = 432 Ω

Whatever the rail doesn't drop across the LED lands on the resistor:

    Vr = Vrail − Vf
    R  = Vr / Itarget

Target 3 mA. An indicator only needs to be visible.

    Vr = 3.3 − 2.0          = 1.3 V
    R  = 1.3 / 0.003        = 433.3 Ω  →  E96: 432 Ω
    I  = 1.3 / 432          = 3.01 mA
    P  = (3.01 mA)² × 432 Ω = 3.9 mW   (vs 250 mW rating)

The datasheet gives only 1.5 V min and 2.4 V max, so I used 2.0 V and checked both ends:

    Vf = 1.5 V  →  4.17 mA
    Vf = 2.4 V  →  2.08 mA

Both are well under 20 mA.

---

## Trace width

IPC-2221, external layer, 10 °C rise:

    I = k × ΔT^0.44 × A^0.725      k = 0.048, A in mil²

The stackup uses 1.4 mil copper (1 oz). For the 0.254 mm (10 mil) traces on the board:

    A = 10 × 1.4             = 14.0 mil²
    I = 0.048 × 10^0.44 × 14^0.725
      = 0.048 × 2.754 × 6.78
      = 0.90 A

Solving the other way for the design current:

    A = 6.26 mil²   →   width = 6.26 / 1.4 = 4.5 mil = 0.11 mm
    temperature rise of a 0.254 mm trace at 0.5 A = 2.7 °C

- 0.254 mm carries about 1.8× the design current.
- The 3V3 side mostly runs through the pour anyway. The 5V side (J1 → C1 → U1) is all trace, and that's where the check matters.

---

## Design rules (as built)

| Rule | Value |
|---|---|
| Clearance | 0.254 mm |
| Trace width | 0.254 mm (min = max = preferred) |
| Via | 1.27 mm diameter, 0.711 mm hole |
| Hole size | 0.025 – 2.54 mm |
| Min annular ring | 0.076 mm (actual: header 0.25 mm, via 0.28 mm) |
| Solder mask expansion | 0 mm; **+0.2 mm on C1/C2** |
| Min solder mask sliver | 0.254 mm |
| Silk to solder mask | 0.15 mm |
| Polygon connect | 4-spoke relief, 0.127 mm spokes; **direct connect on U1**; vias direct |
| Stackup | 1.4 mil Cu / 0.32 mm FR-4 / 1.4 mil Cu |

Notes on the two non-default rules:

- **Direct connect on U1.** Relief spokes would restrict exactly the heat path the pour is there for. The cost is that the tab is harder to hand-solder.
- **+0.2 mm mask on C1/C2.** The footprint's pads are 0.24 mm apart, below the 0.254 mm sliver minimum. The extra expansion makes the two openings overlap by 0.16 mm, so there's no sliver left to violate the rule. The trade-off is no mask dam between the pads, which raises bridging risk when soldering.
