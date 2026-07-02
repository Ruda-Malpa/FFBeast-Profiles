# DCS Mirage F1 — FFBeast Commander tuning notes
Profile file: `DCS_Mirage_F1_FFBeast_Profile.json`

## Aircraft/control-system classification
- **Simulator/module:** DCS: Mirage F1 by Aerges. Baseline metadata uses `flight_data_source = 1` and `flight_module = "Mirage-F1"`; verify the exact Commander-detected string in your install.
- **Control system:** hydraulically powered / electro-hydraulic flight-control system with artificial feel and flying aids, not a manual cable-loaded warbird and not a modern full-FBW force law. **[CONFIRMED]** from DCS/Aerges description and Mirage F1 manual schematics.
- **Profile class:** non-FBW / hydraulically boosted analog jet. Primary centering is fixed-gradient `simple_spring`; pitch gets a modest `g_force_spring`; airspeed spring is a tiny q-feel trace only, not the main force source.

## Real data anchors and evidence tags
| Anchor | Value used | Tag | How used |
|---|---:|---|---|
| FFBeast rig max grip force | 14 kg | [CONFIRMED project hardware] | Clamp check: `felt_kg = min(requested_kg, 14)` |
| DCS Mirage F1 flight-control system | Normal electro-hydraulic mode with degraded modes | [CONFIRMED] | Aircraft class: boosted/artificial-feel, not raw airspeed-loaded |
| Aerges FFB implementation | Simulates artificial feel system / flight controls / flying aids | [CONFIRMED] | Avoids heavy extra DirectX force and avoids overdone vibration |
| Mirage F1 manual hydraulic schematic | Pitch, aileron, spoiler, and rudder servo-controls | [CONFIRMED] | Supports boosted/servo-control classification |
| Load factor limit | +7.2 / -3 subsonic, +6 / -3 supersonic | [CONFIRMED] | `over_g_shaker` starts before +7.2; pitch g-wall stays under 14 kg until abusive pulls |
| Maximum indicated speed | 700 kt 0–20,000 ft; 750 kt above 20,000 ft; M 2.10 | [CONFIRMED] | Airspeed check points and overspeed-test checklist |
| Stick-force-per-G | not found in accessible primary sources | [UNKNOWN] | `g_force_spring` slope is conservative [INFERRED], not a claimed real Mirage F1 number |

## Per-axis force rationale
### Elevator / pitch
- `simple_spring`: full-aft target **5.5 kg** with a 0.9 kg near-center breakout. **[INFERRED]** Conservative artificial-feel baseline, heavier than F-5E but not a hard wall.
- `g_force_spring`: roughly 0.5 kg at 1 g, 2.2 kg at 4 g, 6.2 kg at 7.2 g, 8 kg at 8 g. **[INFERRED]** Adds the missing maneuver-load cue while keeping full-aft + 7.2 g below 14 kg.
- `airspeed_spring`: only **1.0 kg at 1400 km/h**. **[INFERRED]** Trace high-q cue for an analog artificial-feel aircraft, deliberately not a warbird-style airspeed wall.
- No `aoa_spring`: the Mirage F1 has no FBW alpha-limit stick wall in this profile. High-AoA is represented by DCS generic shaker plus a small `aoa_shaker`. **[SIM]**

### Ailerons / roll
- `simple_spring`: full-roll target **3.2 kg** with 0.55 kg breakout. **[INFERRED]** Lighter than pitch for normal fighter harmony, with hydraulic/artificial-feel damping.
- `airspeed_spring`: only **0.7 kg at 1400 km/h**. **[INFERRED]** Slight high-speed roll firmness, no saturation.
- `over_g_shaker` and `aoa_shaker` provide awareness cues without changing roll centering. **[SIM]**

### Rudder / yaw
- `simple_spring`: full-pedal target **10 kg** with 2.4 kg breakout. **[INFERRED]** Coarse, heavy pedals without clamping on a 14 kg rig.
- Damping is high enough to avoid twitchy pedals: static 50 / dynamic 95. **[INFERRED]**

## Force-budget table
Values are **requested kg / felt kg** on a 14 kg FFBeast rig. Pitch values assume full aft stick plus the listed g; roll and rudder are full-command checks.

| Condition | Speed km/h | G | Pitch req/felt kg | Roll req/felt kg | Rudder req/felt kg | Notes |
|---|---:|---:|---:|---:|---:|---|
| Taxi / ground check | 0 | 1.0 | 6.00 / 6.00 | 3.20 / 3.20 | 10.00 / 10.00 | full-command check; shakers scale with ground roll only |
| Takeoff rotation, ~160 kt / 296 km/h | 296 | 1.0 | 6.05 / 6.05 | 3.23 / 3.23 | 10.00 / 10.00 | full-command budget, normal rotation should use much less |
| Cruise, ~450 kt / 833 km/h | 833 | 1.0 | 6.35 / 6.35 | 3.45 / 3.45 | 10.00 / 10.00 | full-command budget |
| 4 g combat pull, ~450 kt / 833 km/h | 833 | 4.0 | 8.05 / 8.05 | 3.45 / 3.45 | 10.00 / 10.00 | full aft pitch + g feel |
| High speed, 700 kt / 1296 km/h | 1296 | 1.0 | 6.86 / 6.86 | 3.80 / 3.80 | 10.00 / 10.00 | near low-altitude manual speed limit |
| Structural limit, 500 kt / 926 km/h, 7.2 g | 926 | 7.2 | 12.14 / 12.14 | 3.51 / 3.51 | 10.00 / 10.00 | full aft pitch + g wall cue |
| Over-G warning, 500 kt / 926 km/h, 8.0 g | 926 | 8.0 | 13.94 / 13.94 | 3.51 / 3.51 | 10.00 / 10.00 | still just below 14 kg clamp |
| Overspeed check, 750 kt / 1389 km/h | 1389 | 1.0 | 6.99 / 6.99 | 3.89 / 3.89 | 10.00 / 10.00 | above 20,000 ft manual speed limit |

No primary axis intentionally clamps in normal flight. The highest listed pitch case is just below the 14 kg clamp so the aircraft still has a readable gradient instead of a wall.

## Shaker/jolt layer
- `generic_shaker`: modest, because DCS already supplies precomputed vibration channels and Aerges intentionally models FFB around the artificial-feel system.
- `aoa_shaker`: small high-AoA/buffet cue. Reduce first if stall buffet feels artificial or double-counted.
- `over_g_shaker`: starts around 5.5 g and becomes obvious near 7.2 g.
- Gear, flaps, airbrake, landing, damage, gunfire, weapon release, countermeasure effects are intentionally low-to-moderate.

## Required manual setup
1. FFBeast Setup: set **Max force available = your true measured max**, here **14 kg**. Do not use this as a volume knob.
2. FFBeast Commander: DCS telemetry source = **DCS** (`flight_data_source = 1`).
3. Verify the detected module name in Commander. The profile uses partial string `Mirage-F1`; if auto-load fails, copy the exact detected module string, likely one of the Mirage F1 variant names.
4. Keep DCS in-game stick-shake/FFB extras conservative if they double-count the FFBeast shakers.
5. Dynamic Motion Range: normally **off for DCS**; it is mainly for MSFS / IL-2 / War Thunder input-scaling compensation.

## Flight-test checklist
- Cold/dark and engine start: confirm no runaway force or oscillation. Some force before hydraulic pressure is a sim/profile limitation because hydraulic-pressure telemetry is not used.
- Taxi: check on-ground rumble; reduce `on_ground_shaker` if it masks steering feel.
- Takeoff: rotate gently; force should be firm but not heavy.
- Cruise 350–500 kt: roll should be lighter than pitch; pitch should not feel like a warbird airspeed wall.
- 3–5 g turns: pitch force should build clearly through `g_force_spring`.
- 6–7.2 g pull: `over_g_shaker` should warn before the real +7.2 g subsonic limit.
- High AoA: buffet should be present but not violent; if DCS generic shaker already does this well, reduce `aoa_shaker` by 30–50%.
- Airbrake/flaps/gear: feel only secondary rumble, not a centering-force change.

## Known uncertainties and recommended tuning
- **Real Mirage F1 stick-force-per-G was not found** in the accessible sources used here. Pitch G spring slope is therefore **[INFERRED]**. If you find a flight-test or manual value, replace the `g_force_spring` curve directly in kg.
- If the stick feels too light in combat turns, increase `g_force_spring` by +10–15% first, not the airspeed spring.
- If the stick feels too heavy all the time, reduce elevator `simple_spring` full-aft from 5.5 kg toward 4.5 kg.
- If roll feels sluggish, reduce aileron `smart_damper.dynamic_dampening` from 70 toward 55 before reducing spring force.
- If native Aerges FFB feels better on your rig, alternate experiment: set `direct_x.force` on pitch/roll to 30–40 and reduce or disable `simple_spring`/`g_force_spring`; do not stack both full strength.
