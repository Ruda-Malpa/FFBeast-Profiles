# FFBeast Commander Profile Notes — IL-2 Korea F-80C-10 Shooting Star

Profile file: `IL2K_F-80C-10_FFBeast_Profile.json`  
Target sim: **IL-2 Korea**  
Target module string: **`F-80C-10`**  
Reference hardware: **FFBeast base, ~15 cm extension, true max grip force 14 kg**

## 1. Aircraft/control-system classification

**Aircraft:** Lockheed F-80C-10 Shooting Star.  
**Closest measured public force data used:** Lockheed **P-80A** NACA flight-test reports, because I did not find a public F-80C-10 stick-force chart with full speed/G curves.

| Axis | Real-system classification | Profile model | Evidence tag |
|---|---|---|---|
| Elevator | Direct/spring-tab elevator; measured P-80A report flags excessive maneuvering elevator control forces | `g_force_spring` is co-primary; small `airspeed_spring`; modest `simple_spring` friction/breakout | Control problem [CONFIRMED], exact kg/G [INFERRED] |
| Aileron | Ailerons have no aerodynamic balance but use 15:1 hydraulic boost; 4.5 lb through-neutral friction | Boosted feel: `simple_spring` with confirmed breakout; only trace `airspeed_spring` | [CONFIRMED] + full-travel force [INFERRED] |
| Rudder | Rudder has measured 8 lb friction and centering spring | Heavy/coarse `simple_spring`, no speed-saturating wall | [CONFIRMED] + full-pedal force [INFERRED] |

## 2. Real data anchors used

- **P-80A longitudinal report:** tests up to **530 mph IAS** at low altitude and **350 mph IAS** at high altitude; the airplane mostly met Army flying-quality requirements, except for **excessive elevator control forces in maneuvering flight** and insufficient low-speed longitudinal trim authority. **[CONFIRMED]**
- **P-80A lateral/directional report:** tests up to **494 mph IAS** at low altitude and **378 mph IAS** at high altitude; most requirements were met except negative lateral stability with wing-tip tanks installed. **[CONFIRMED]**
- **Ailerons:** piano-hinged on upper wing surface, **no aerodynamic balance**, forces reduced by **15:1 hydraulic boost**, measured **4.5 lb** through neutral with boost operating. Conversion: `4.5 × 0.4536 = 2.04 kg`. **[CONFIRMED]**
- **Rudder:** measured **8 lb** friction at the pilot pressure point, and a centering spring helped return the control to neutral. Conversion: `8 × 0.4536 = 3.63 kg`. **[CONFIRMED]**
- **Exact F-80C-10 elevator stick-force-per-G:** not found in public sources during this pass. The profile therefore uses a conservative, heavy early-jet inferred value of **~3.6 kg/G above 1 g** so the high-speed/high-G corner intentionally approaches the 14 kg device clamp. **[INFERRED]**

## 3. Profile authoring style

This is a **literal-kg** profile for normal-envelope feel, with one deliberate high-G clamp:

- `total_effect_strength = 100` on spring effects, so curve outputs are kg.
- Normal pitch/roll/rudder forces remain readable and below clamp.
- Elevator `g_force_spring` intentionally reaches the 14 kg rig limit near ~5 g. This is the profile’s “excessive maneuvering force / stop pulling” cue, not an everyday centering force.
- Aileron and rudder are not speed-saturated; they use mechanical/hydraulic feel and confirmed friction anchors.

## 4. Per-axis rationale

### Elevator / pitch

Primary design choice: **G-force loading, not a big airspeed spring.** The NACA P-80A report calls out excessive elevator maneuver forces, which is best represented as force rising with normal acceleration. The profile uses:

- `g_force_spring`: ~3.6 kg per G above 1 g, reaching 14 kg requested at 5 g. **[INFERRED]**
- `airspeed_spring`: only 3.0 kg at 900 km/h IAS, for light speed/static stability feel. **[INFERRED]/[SIM]**
- `simple_spring`: 0.8 kg near-center breakout, 4.5 kg at full deflection. **[INFERRED]**
- `elevator_weight`: 15, modest residual droop/weight cue. **[INFERRED]**
- `aoa_shaker`: stall/buffet cue only; no `aoa_spring` wall, because the F-80 is not an alpha-command FBW jet. **[SIM]**

### Ailerons / roll

The real ailerons are mechanically unbalanced but hydraulically boosted 15:1, with measured 4.5 lb through-neutral friction. Therefore the profile avoids a manual-warbird airspeed wall:

- `simple_spring`: near-center ±2.05 kg, matching the measured 4.5 lb friction. **[CONFIRMED]**
- Full lateral force rises only to 4 kg. **[INFERRED]**
- `airspeed_spring`: trace 0.8 kg at 900 km/h to keep some q cue without overriding boost. **[SIM]**
- Damping is deliberately moderate-high for early hydraulic/boosted feel. **[INFERRED]**

### Rudder

The rudder has measured 8 lb friction and a centering spring, so it should feel heavier/coarser than the ailerons:

- `simple_spring`: near-center ±3.6 kg, matching measured 8 lb friction. **[CONFIRMED]**
- Full pedal request 13 kg, just under the 14 kg rig clamp. **[INFERRED]**
- `trimmer = 0` on rudder. **[SIM]**

### Shakers and jolts

- `gunfire_shaker`: moderate, nose-gun feel for six .50 cal guns. **[SIM]**
- `aoa_shaker`: stall warning/buffet. **[SIM]**
- `over_g_shaker`: starts at ~4 g and rises through 5–8 g. **[SIM]**
- `airbrake_shaker`, `flaps_shaker`, `gear_shaker`, `landing_jolt`: secondary and conservative. **[SIM]**
- `afterburner_shaker` and `countermeasuers_jolt`: disabled; F-80C has no afterburner or modern countermeasures. **[CONFIRMED]/[SIM]**

## 5. Force-budget validation

The numbers below are **spring-force requests only**. Vibrations/jolts are not included.  
Felt force assumes FFBeast Setup **Max Force Available = 14 kg**.

| Condition | IAS km/h | G | Deflection used | Elevator requested/felt | Aileron requested/felt | Rudder requested/felt | Clamp? |
|---|---:|---:|---:|---:|---:|---:|---|
| Taxi / controls check | 0 | 1.0 | 10% | 0.99 / 0.99 kg | 2.13 / 2.13 kg | 4.02 / 4.02 kg | No |
| Takeoff roll / rotation | 210 | 1.0 | 25% | 1.57 / 1.57 kg | 2.35 / 2.35 kg | 4.80 / 4.80 kg | No |
| Climb | 380 | 1.5 | 35% | 4.14 / 4.14 kg | 2.64 / 2.64 kg | 5.76 / 5.76 kg | No |
| Cruise | 650 | 1.0 | 25% | 2.97 / 2.97 kg | 2.72 / 2.72 kg | 4.80 / 4.80 kg | No |
| Combat turn | 650 | 3.0 | 55% | 11.37 / 11.37 kg | 3.34 / 3.34 kg | 7.76 / 7.76 kg | No |
| High-speed 1 g | 850 | 1.0 | 20% | 3.95 / 3.95 kg | 2.96 / 2.96 kg | 4.54 / 4.54 kg | No |
| High-speed pull | 850 | 4.0 | 60% | 16.28 / 14.00 kg | 3.76 / 3.76 kg | 8.32 / 8.32 kg | Elevator intentional clamp |
| Near over-G | 750 | 5.0 | 70% | 19.29 / 14.00 kg | 3.84 / 3.84 kg | 9.44 / 9.44 kg | Elevator intentional clamp |
| Stall / buffet approach | 230 | 1.2 | 40% | 2.92 / 2.92 kg | 2.65 / 2.65 kg | 6.24 / 6.24 kg | No |

## 6. Required manual setup

1. In **FFBeast Setup**, set **Max Force Available = your true measured max**, here **14 kg**. Do not use this as a gain/volume knob.
2. In FFBeast Commander / simulator configuration, use **IL-2 data source**. The profile metadata uses `flight_data_source = 2`.
3. Confirm the live module name in the Telemetry Debug tab. If IL-2 reports a different string, rename `metadata.flight_module` from `F-80C-10` to the exact reported substring.
4. Enable **Dynamic Motion Range** for IL-2. This is important because IL-2/WT/MSFS can change effective stick travel with speed.
5. Confirm telemetry airspeed is effectively IAS/EAS in km/h. If a future module feeds TAS, the `airspeed_spring` will over-read at altitude.
6. Reduce any in-game camera/stick shake that double-counts stall, ground, or gunfire vibration.

## 7. Flight-test checklist

- **Ground:** stick should not feel dead, but aileron and rudder should have noticeable breakout. If roll feels notchy, lower the aileron ±3% points from 2.05 kg to ~1.5 kg.
- **Takeoff:** pitch should be controllable and not heavy during rotation. If rotation feels sluggish, lower elevator `simple_spring` full-deflection values by 10–15%.
- **Cruise trim:** at 600–700 km/h and 1 g, elevator should feel around 3 kg for moderate displacement; it should not be a wall.
- **3 g turn:** pitch should become distinctly heavy but still controllable, around 11 kg requested at the test condition.
- **High-speed pull:** at ~850 km/h and 4 g, elevator clamp is intentional. It should warn against hauling the stick at high q.
- **Stall approach:** buffet should appear through `aoa_shaker`; centering force should not become an artificial alpha wall.
- **Gunfire:** six-gun vibration should be present but not dominate aim. Reduce `gunfire_shaker` by 25% if it scatters the sight excessively.

## 8. Known uncertainties and suggested tuning

- **F-80C-10 exact elevator kg/G remains [UNKNOWN].** The profile uses P-80A flight-test criticism plus early-jet handling logic to infer a heavy maneuver gradient.
- **If pitch is too heavy:** reduce `elevator.g_force_spring.curve1` outputs by 15–25%. Keep the shape; scale values down.
- **If pitch is too light in turns:** increase `elevator.g_force_spring` to 4.0–4.5 kg/G above 1 g, but re-check the high-speed/high-G clamp.
- **If roll is too sticky:** reduce the aileron near-center ±3% points from ±2.05 kg toward ±1.5 kg.
- **If rudder pedals are too hard:** reduce the rudder near-center ±2% points from ±3.6 kg toward ±2.8 kg and full-pedal from 13 kg toward 10–11 kg.
- **If stall buffet double-counts IL-2 native cues:** reduce `aoa_shaker.total_effect_strength` on elevator and ailerons by 30–50%.

## 9. Source list

1. NACA/NASA NTRS, *Flight Measurements of the Flying Qualities of a Lockheed P-80A Airplane (Army No. 44-85099): Longitudinal-Stability and -Control Characteristics*, NACA-RM-A7G01, 1947. https://ntrs.nasa.gov/citations/20030064139
2. NACA/NASA NTRS, *Flight Measurements of the Flying Qualities of a Lockheed P-80A Airplane (Army No. 44-85099): Lateral- and Directional-Stability and Control Characteristics*, NACA-RM-A7J24, 1947. https://ntrs.nasa.gov/citations/20030063229
3. UNT Digital Library page image/OCR for NACA-RM-A7J24 page 3, aileron boost/friction and rudder friction details. https://digital.library.unt.edu/ark:/67531/metadc63775/m1/3/
4. FFBeast Commander effects documentation: https://ffbeast.github.io/docs/en/ffbeast_commander_effects.html
5. FFBeast Setup effects documentation: https://ffbeast.github.io/docs/en/ffbeast_setup_effects.html
6. Project rulebook: `FFBeast_Profile_Design_Instructions.md`.
