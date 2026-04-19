# Datsun 280ZX Speeduino L28E ECU Swap

Complete technical documentation for retrofitting a Datsun 280ZX with a Speeduino-based engine control unit (ECU) driving a Nissan L28E engine.

## Overview

This repository contains comprehensive wiring diagrams, pin mappings, sensor calibration procedures, and a complete tuning workflow for integrating Speeduino v0.4.4 (Arduino Mega 2560) into a classic Datsun 280ZX with L28E engine management.

**Target Configuration:**
- **ECU Platform:** Speeduino v0.4.4 on Arduino Mega 2560
- **Engine:** Nissan L28E (6-cylinder, naturally aspirated)
- **Ignition:** Wasted spark configuration with EDIS6 coil pack
- **Fuel System:** Sequential injection (optional; batch fire acceptable for NA)
- **Sensor Feedback:** Wideband O2, MAP, IAT, TPS, CLT, CKP, CMP

## Contents

### [`SPEEDUINO_L28_PIN_REFERENCE.md`](SPEEDUINO_L28_PIN_REFERENCE.md)
The complete technical reference covering:

- **Pin Mappings** — All 48 Speeduino pins with function, control registers, and physical connections
- **Ignition System** — Wasted spark fire pattern (1→5→3→6→2→4), EDIS6 coil pack wiring, primary/secondary circuits
- **Fuel System** — Sequential injector timing, pump prime sequence, pressure regulation (43–45 PSI NA)
- **Analog Sensors** — CKP (crank trigger), MAP (0–250 kPa), IAT, TPS (0.5–4.5V range), CLT, wideband O2
- **Digital I/O** — Relay logic for fuel pump, cooling fan, check engine light
- **Wiring Sequence** — 8-phase installation from ground star-point through bench testing and tuning
- **Calibration Tables** — Sensor ADC ranges, TunerStudio configuration, VE table starting points
- **Troubleshooting** — 6 common failure modes with root causes and solutions

### [`speeduino_l28_wiring.html`](speeduino_l28_wiring.html)
Visual wiring reference and schematic diagrams for rapid lookup.

### [`CLAUDE.md`](CLAUDE.md)
Development and documentation maintenance guide for future contributors.

## Quick Start

### Prerequisites

- Arduino Mega 2560 with Speeduino bootloader
- Nissan L28E engine (or similar 6-cyl. distributor-less target)
- 36-1 toothed crank trigger wheel + Hall sensor
- EDIS6 ignition module (or 6x smart coil packs)
- Walbro 255 LPH fuel pump
- 6x high-impedance fuel injectors (12Ω+)
- Sensors: CKP, MAP, IAT, TPS, CLT (CMP optional for sequential)
- Wideband O2 sensor + controller
- TunerStudio software for ECU tuning

### Installation Path

1. **Read the wiring sequence** (Phases 1–7 in `SPEEDUINO_L28_PIN_REFERENCE.md`)
2. **Prepare hardware** — Mount trigger wheel, install Speeduino, prepare ground star-point
3. **Cable all systems** — Ignition, fuel, sensors in prescribed order (critical for debugging)
4. **Verify on bench** — Multimeter check every voltage and sensor input before powering engine
5. **Load base tune** — Use Speeduino community L28 base configuration (~30–40% VE table)
6. **First start** — Cold crank, prime fuel, monitor TunerStudio live data
7. **Tune for idle** — Adjust VE table ±5–10% per iteration; target lambda 1.0 with wideband
8. **Datalog & iterate** — CSV export, VE Analyze tool, refine by RPM/MAP load cells

## Critical Design Principles

### Ground Star-Point (Non-Negotiable)
All sensor grounds and power returns converge at a single point on the engine block. Do not distribute ground returns—it causes sporadic misfire, trigger errors, and sensor noise.

### Signal Isolation
- **Ignition wiring:** 16 AWG silicone, twisted pair with ground, minimum 6" separation from sensor cable
- **CKP trigger:** Optoisolated (TLP521) to reject electrical noise from L28 block
- **Sensor wiring:** 22 AWG shielded, capacitor 0.1µF at ECU input

### Calibration Authority
Key operational constraints (immutable without hardware change):
| Parameter | Value | Sensor |
|-----------|-------|--------|
| MAP range | 0–250 kPa | Freescale MPX4250 |
| TPS range | 0.5–4.5V | 3-pin pot on throttle |
| CKP trigger | 36-1 wheel, 2–3mm gap | Hall sensor |
| Fuel pressure (NA) | 43–45 PSI | Regulator on Walbro 255 |
| Ignition baseline | 5° BTDC | Safe start; optimize via log |
| Lambda target | 1.0 stoichiometric | Wideband feedback during tune |

## Wiring Checklist

- [ ] Ground star-point: 4 AWG from battery, motor block, chassis, sensors (all at one M8 terminal)
- [ ] Speeduino power: 4 AWG red → 60A fuse → 5-pin relay → Vin + GND
- [ ] Ignition (EDIS6): Pins 24, 26, 28, 30, 32, 34 → Primaries A, B, C, D, (E), (F)
- [ ] Fuel injectors: Pins 11–13, 44–46 → Ground sides of EV6 connectors
- [ ] Fuel pump relay: Speeduino D7 → 5-pin 12V relay → Walbro 255
- [ ] CKP trigger: Hall sensor → Optoacoupler → Pin 20 (INT4) + 1kΩ pull-up
- [ ] MAP sensor: Freescale MPX4250 → A1 with 0.1µF bypass
- [ ] IAT, TPS, CLT, CMP: A2, A3, A4, A5 (A5 optional) with 0.1µF local caps

## Tuning Workflow

1. **Bench verification** (no engine running)
   - Multimeter: +12–13V on Vin, all sensor ADCs read in range
   - TunerStudio: CKP pin 20 blinks when crank is rotated manually
   - Fuel pressure: 43–45 PSI at rail (key on, pump priming)

2. **Cold crank start** (engine off, 1–2 sec)
   - Confirm spark at plugs (gap check first)
   - Confirm fuel mist from injectors
   - Listen for brief combustion, shut off immediately

3. **Idle stabilization** (5 min ramp)
   - 800 → 1000 → 1500 → 2000 RPM, monitor temperature rise
   - Live adjust VE table if running rich (black smoke) or lean (roughness)
   - Target: smooth idle, lambda 1.0 with wideband

4. **Datalog & iterate**
   - Record 10–15 min of normal driving (idle, cruise, light accel)
   - Export CSV, analyze with VE Analyze tool
   - Adjust cells that are ±10% from stoichiometric
   - Retest, log, refine until consistent lambda

## Troubleshooting

| Symptom | Root Cause | Fix |
|---------|-----------|-----|
| No spark | CKP not reading, ignition polarity reversed, coil unpowered | Verify INT4 blinks; check EDIS6 +12V and primary wiring polarity |
| Rough idle | VE table too lean; clogged injector; vacuum leak | Raise VE 10%; ultrasonic clean injectors; pressure test intake |
| Won't start | Fuel pump not priming; low pressure; injectors not pulsing | Verify D7 relay activates; check 43 PSI; scope injector pins |
| False wideband reading | Sensor unplugged; serial baudrate mismatch; sensor fouled | Reseat AEM UEGO; verify 9600 baud in TunerStudio; replace sensor if soot |
| Trigger errors (multiple) | Ground loop; EMI on CKP line; trigger wheel off-balance | Verify star-point grounding; add optoacoupler if none; rebalance wheel |

## Community & Resources

- **Speeduino Official:** https://speeduino.com/
- **Speeduino Forums:** https://speeduino.com/forum/
- **Nissan L-Series Community:** Datsun Roadster/Fairlady forums (vintage car communities)
- **TunerStudio:** https://www.tunerstudio.com/

## License

This documentation is provided as-is for educational and personal use. Speeduino is open-source under the GPLv3 license.

## Contributing

For improvements, corrections, or additional sensor configurations, please ensure:
1. All pin assignments match Speeduino v0.4.4 ATmega2560 pinout
2. Calibration ranges are validated with actual hardware
3. Wiring diagrams are tested on a running engine
4. Troubleshooting entries include symptom → root cause → solution

---

**Last Updated:** 2026-04-19  
**Version:** 1.0  
**Target Hardware:** Datsun 280ZX + Speeduino v0.4.4 + L28E NA
