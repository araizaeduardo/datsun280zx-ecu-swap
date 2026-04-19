# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains comprehensive technical documentation for retrofitting a Datsun 280ZX with a Speeduino-based ECU (engine control unit) driving a Nissan L28E engine. The focus is on wiring, sensor integration, and tuning configuration.

**Key Project Details:**
- **ECU Platform:** Speeduino v0.4.4 running on Arduino Mega 2560
- **Engine:** Nissan L28E (6-cylinder, naturally aspirated)
- **Ignition System:** Wasted spark configuration with EDIS6 coil pack
- **Fuel System:** Sequential injection with optional wideband O2 feedback
- **Target State:** Drive-ready L28 swap with full engine management

## Repository Structure

```
datsun280zx-ecu-swap/
├── SPEEDUINO_L28_PIN_REFERENCE.md    # Complete pin mapping & wiring guide
└── speeduino_l28_wiring.html         # Visual wiring reference (generated)
```

The main reference document is exhaustively detailed—treat it as the authority for all pin assignments, sensor connections, and calibration procedures.

## Critical Wiring Principles

Before making any changes to documentation, understand these non-negotiable design constraints:

1. **Ground Star-Point Architecture:** All sensor grounds and power returns must converge at a single star-point on the engine block (not distributed). This is the single most critical reliability factor. Any documentation change that muddies this requirement will cause failures.

2. **Signal Isolation:** 
   - Ignition wiring (16 AWG silicone, twisted pair with ground) must be physically separated 6+ inches from sensor wiring (22 AWG shielded)
   - CKP (crank position) signal must be optoisolated due to L28's electrical noise environment

3. **Sequential vs. Batch Firing:** CMP (cam position) sensor on pin 21 (INT5) is optional. Documentation frequently contrasts "with CMP = sequential precision" vs. "without CMP = batch fire (acceptable for NA)". Preserve this distinction in any edits.

4. **Sensor Voltage Conditioning:**
   - Analog sensors (MAP, IAT, TPS, CLT, CMP) require 0.1µF bypass capacitors at the ECU input
   - No analog sensor may draw from the internal +5V rail; use locally regulated 7805
   - High-impedance injectors (12Ω+) on pins 11-13, 44-46; never swap order

## Key Calibration References

These values appear throughout the docs and define the tuning envelope:

- **MAP Range:** 0-250 kPa (Freescale MPX4250), ADC 0-1023 → kPa 0-250 linear
- **TPS Range:** Idle ~0.5V (ADC ~102), WOT ~4.5V (ADC ~921); never saturate to 5V
- **CKP Trigger:** 36-1 toothed wheel, 2-3mm gap, Hall sensor on damper
- **Fuel Pressure (NA):** 43-45 PSI (regulated via Walbro 255 pump)
- **Ignition Advance (baseline):** 5° BTDC (safe starting point; optimize via datalog)
- **Lambda Target:** 1.0 stoichiometric (achieved via VE table iteration with wideband feedback)

## Tuning Workflow

The document prescribes a specific sequence for achieving a running engine, listed as "FASE 8" (phases 1-7 are wiring):

1. **Verify sensor inputs** (ADC values make sense) before attempting cranking
2. **Load base tune** from Speeduino community (conservative VE table ~30-40%)
3. **Achieve ignition** (check for spark at plugs)
4. **Prime fuel system** (listen for pump, measure 43-45 PSI at rail)
5. **Start cold** (1-2 seconds only; listen for combustion)
6. **Idle stabilization** (gradually ramp RPM over 5 min; monitor exhaust)
7. **Live tuning** (adjust VE table ±5-10% per iteration based on wideband lambda)
8. **Datalogging & iteration** (CSV export, VE Analyze tool, refine by RPM/load cells)

If any documentation change affects this sequence, flag the impact to the user.

## When to Suggest Changes

- **Pin assignments** are hardware-locked and cannot change without major rework; treat as immutable
- **Calibration tables** (MAP kPa limits, TPS voltage ranges) should only change if there is a documented hardware revision (e.g., sensor model upgrade)
- **Phase sequence** can be reorganized for clarity, but preserve the logical dependency order (power → grounding → cabling → sensors → firmware → startup)
- **Troubleshooting content** benefits from real-world failure modes; add new entries only with clear symptom → root cause → solution logic

## Testing & Validation

There are no unit tests or CI pipelines in this repository—validation is hardware-centric:

- **Bench testing:** Multimeter verification of voltages at each sensor/output pin before power-on
- **Cold crank testing:** Verifying trigger signal (INT4 pin 20 blinking in TunerStudio) with motor not running
- **Hot running:** Real engine startup and idle stability are the ultimate validators

When reviewing changes, always ask: "Does this change affect the critical path to a first start?"

## Documentation Maintenance Notes

- **Language:** English + Spanish terminology common (e.g., "tierra" = ground, "bobina" = coil); preserve for community reach
- **Version tracking:** Documents are dated and versioned; update `Last Update` and `Version` fields at the end of the main reference if content changes
- **Community references:** The docs link to Speeduino official forums and Nissan L-Series communities; preserve these attribution links
- **ASCII diagrams:** Wiring flow diagrams are ASCII art; test rendering in monospace after edits to ensure alignment
