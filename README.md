# 10kW Dual Active Bridge DC-DC Converter — PCB

[![License: CERN-OHL-S v2.0](https://img.shields.io/badge/License-CERN--OHL--S%20v2.0-blue.svg)](https://ohwr.org/cern_ohl_s_v2.txt)
[![KiCad](https://img.shields.io/badge/Made%20with-KiCad-blue)](https://www.kicad.org/)

<img src="https://raw.githubusercontent.com/NovaPrime18/DAB_DCDC/main/Pictures/DAB9T.png" width="100%">

Hardware files for a high-power, high-frequency DC-DC converter PCB, developed as a
prototype module for a modular 500kW off-grid EV DC fast charger. The board implements
a Dual Active Bridge (DAB) topology with a planar transformer, isolated gate drivers and
sensing, and on-board water/air cooling provisions for the power stage.

This board is a single prototype module. Multiple modules are intended to be run in
parallel and/or series to scale up to the full 500kW system.

---

## Key specifications

| Parameter | Value |
|---|---|
| Topology | Dual Active Bridge (DAB) |
| Input voltage | 500–800V DC |
| Output voltage | 250–1000V DC (peak efficiency at 666V) |
| Output power | ~10kW per module |
| Output current | Primary 15A (22A peak), Secondary 17A (25A peak) |
| Switching frequency | 100kHz |
| Isolation | Galvanic, via transformer (10,000V dielectric strength primary/secondary) |
| Transformer | Payton Planar, 24:20 turns ratio, 35µH total leakage inductance |
| Control | TI controlCARD via CAN bus, phase-shift power regulation |
| PCB | 6 layers, 70µm (2oz) copper, 2mm FR4 |
| Secondary switches | Microchip MSC035SMA170B4 SiC MOSFETs (1700V) |
| Clearance / creepage | 3mm (primary) / 3.5mm (secondary), per IPC-2221 / IEC-60664 |
| Voltage derating | 1.7x on HV secondary components |
| Cooling | Fin-stack air-cooled heatsinks or aluminum liquid-cooling plate (SiCFETs + transformer) |
| Connectors | XT30 for HV in/out (contactors are off-board) |

## Board design notes

- All components are placed on the **top side only** for simpler, cheaper assembly; the
  transformer sits in the middle and physically supports the board (and the cooling plate,
  when fitted).
- Six copper layers at 70µm are used (instead of the reference design's 4 layers) to reduce
  conduction losses at higher secondary currents.
- HV secondary components are rated to 1700V to handle voltage spikes at the 1000V output
  target, with 1.7x derating.
- DC blocking capacitors are 2280-package, 1700V rated, double-stacked to reach the
  required capacitance. A future revision should move to the 3640 package, which has more
  capacitance per area but does not fit the current footprint.
- Contactors are **not** on the PCB — HV switching is handled externally via XT30
  connectors, allowing higher-voltage contactors to be selected independently of the board.
- Isolated 5V and gate-drive supplies are derived on-board; see "Known issues" below for a
  wiring correction needed for the secondary-side isolated 5V rail.
- Designed and manufactured through JLCPCB (6-layer, double-thick copper, 2mm FR4).

## Design rule summary

These rules were applied during layout and should be followed in any revision:

- **High current:** ~1 via per amp for stitching/heat spreading, no 90° corner traces,
  70µm copper minimum, 6-layer stack-up. Trace rise kept to ≤10°C above ambient at 30A
  (verified with KiCad's trace-width calculator).
- **High frequency:** Ferrite-core planar transformer, short/direct high-frequency traces,
  minimized loop areas to limit EMI and parasitic inductance.
- **High voltage:** Clearance/creepage per IPC-2221 and IEC-60664 (3.5mm creepage at
  1000V), 2x voltage derating where practical, Plastik 70 conformal coating
  (>85kV/mm dielectric strength) on the assembled board.
- **Thermal:** Copper-filled vias, water-cooled or finned aluminum heatsinks on SiCFETs and
  transformer core.

## Known issues / errata (first prototype build)

These were found during assembly/bring-up of the first board and are **not yet fixed on
this design** unless noted — see schematics for full details:

- **eFuse (TPS26400):** Pins 3 and 5 were swapped in the footprint, preventing power-up.
  Fixed on the physical board with bodge wires; needs a schematic/footprint fix in the
  next revision.
- **controlCARD connector:** The standard KiCad/Digikey footprint for TI's connector has
  reversed pin numbering vs. TI's own controlCARD. A separate flip-adapter PCB is required
  as a workaround (included in this repo). The adapter should be ordered in 1mm board
  thickness, not 1.7mm, to avoid breaking the socket pins on connector insertion.
- **Isolated PSU current limit (Rlim):** Original 1k current-limit resistor caused
  overcurrent-protection faults on startup due to 5V rail capacitance. Change to 270R.
- **Primary-side sensing pinout:** DS modulator interface pins to the controlCARD were
  routed to pins that don't support DS modulators; rerouted with magnet wire on the
  prototype and needs correcting in the footprint/routing.
- **Secondary isolated 5V supply:** Originally derived from the "top" isolated supply of
  H2, whose ground floats to 1000V half the switching cycle. Must be re-derived from the
  "bottom" isolated supply of H2 instead.
- **Cooling clearance:** Backside DC capacitors interfere with the aluminum cooling plate;
  stack capacitors on the top side instead. The transformer cutout also needs enlarging —
  it's currently a tight fit.

⚠️ Due to the eFuse/pinout bodges above, **this specific assembled prototype should only
be tested at low voltage (LV), not at its rated HV**, until a corrected revision is built.

## Repository contents

- Schematics and PCB layout (KiCad)
- Fabrication outputs (Gerbers / stencil) as ordered from JLCPCB
- controlCARD adapter PCB (flips connector pinout, 90° mount)
- Renders of the assembled board and cooling solutions

## Background

This board was developed as part of an internship thesis evaluating the feasibility of
in-house production of a 500kW off-grid EV DC fast charger, based on and enhanced from a
Texas Instruments DAB reference design. Full design rationale, simulation results
(LTspice/PLECS), and test results are documented in the accompanying thesis — this README
covers only the PCB/hardware side.
