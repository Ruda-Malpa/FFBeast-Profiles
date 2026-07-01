# DCS F4U-1D Corsair — FFBeast Commander profile notes

Files generated:

- `DCS_F4U-1D_FFBeast_profile.json`
- `DCS_F4U-1D_FFBeast_tuning_notes.md`

## Aircraft and control-system classification

Aircraft: **Vought F4U-1D Corsair**, DCS module by Magnitude 3.

Control-system classification:

- **Elevator:** manual / cable / aerodynamically loaded, with trim-tab behavior and documented light elevator forces at moderate speeds. Main FFBeast force is a light `airspeed_spring`, plus `local_aoa` stall lightening, a modest `g_force_spring`, `elevator_weight`, and low propwash. **[CONFIRMED control behavior, INFERRED numeric pitch kg]**
- **Ailerons:** manual / aerodynamically loaded; production F4U-1D test data gives real full-aileron stick-force anchors. Main FFBeast force is a v² `airspeed_spring` calibrated around the 200 KIAS anchor, with a small mechanical `simple_spring`. **[CONFIRMED anchor, SIM/INFERRED curve compromise]**
- **Rudder:** manual / aerodynamically loaded; documented as normal-to-heavy and impossible to obtain full rudder at maximum speed. Main FFBeast force is a heavy v² `airspeed_spring`, plus stronger propwash. **[CONFIRMED behavior, INFERRED numeric kg]**

## Sources and data anchors

1. **FFBeast Commander / Setup semantics.** Commander spring curves are authored as kg when `total_effect_strength = 100`; Setup's *Max force available* must be the true measured force. The live FFBeast docs state that non-FBW aircraft should normally use `airspeed_spring` as the main aerodynamic spring, prop airflow should be on elevator/rudder rather than ailerons, `g_force_spring` reacts to G force, and shakers/jolts use telemetry-dependent curves. Setup docs define *Max force available* as the real measured grip force and explain that Commander asks for kg and the device scales motor power from that value.

2. **F4U-1D DCS module.** Eagle Dynamics lists the DCS product as **DCS: F4U-1D Corsair** and describes the -1D as using the R-2800-8W with water injection, six .50 cal guns, HVARs, bombs/drop tanks, and a 4,000 lb external-load capability.

3. **U.S. Navy F4U-1 production inspection / final flight report, 1944.** The report covers model F4U-1 production inspection trials and includes model F4U-1D airplane No. 50360 in overload fighter loading. It records full-aileron stick forces at **90 KIAS landing: 5.5/7.5 lb**, **150 KIAS clean: 14/15.5 lb**, and **200 KIAS clean: 17/19.5 lb**. Converted with kg = lb × 0.4536, the 200 KIAS anchor is about **7.7/8.9 kg**. The same report says F4U-1 elevator forces were **very light at moderate speeds** but became heavy in high-speed dives, with a definite lightening of force approaching stall in landing condition; it also states the aileron control was adequate and forces reasonably light, and the rudder forces were normal to high, impossible to obtain full rudder at maximum-speed level, with no rudder-force reversals in normal fighter loading.

4. **DCS forum / EAA article reference.** A DCS forum discussion points to the June 1990 EAA *Sport Aviation* article “Ending the Argument,” comparing FG-1D Corsair, P-47D, F6F-5, and P-51D; the poster summarizes the Corsair as having the lightest stick-force-per-G of the four and being described as almost too light. I used this only as qualitative support for a light pitch profile, not as an exact real-number source.

## Main force choices

### Elevator / pitch

- `airspeed_spring`: literal-kg v² curve, **12.5 kg at 600 km/h full deflection**. This keeps the F4U pitch lighter than P-47-style saturating profiles in ordinary cruise, while still making high-speed dives heavy. **[INFERRED]**
- `g_force_spring`: mild g cue: 0 kg at 1 g, 1.0 kg at 3 g, 3.0 kg at 6 g, steeper toward 7.5–8 g. This models the “pull gets heavier” cue without making the Corsair feel like a boosted jet. **[INFERRED/SIM]**
- `local_aoa`: retained at TES 100 for stall/g-break lightening, matching the reported elevator-force lightening as stall is approached. **[CONFIRMED behavior, SIM implementation]**
- `elevator_weight`: TES 45. The production report mentions an 11 lb bungee spring pulling the stick forward when tailwheel is down; FFBeast `elevator_weight` is a simulation proxy, not a kg-calibrated exact replica. **[CONFIRMED behavior, SIM value]**
- `prop_thrust_spring`: small 2.5 kg peak, elevator only. **[INFERRED]**

### Ailerons / roll

- `airspeed_spring`: v² curve calibrated to **6.7 kg at 200 KIAS**, plus `simple_spring` 1.6 kg full-deflection mechanical/breakout component. That yields about **8.3 kg requested at 200 KIAS full deflection**, matching the average of the 17/19.5 lb F4U-1D test points. **[CONFIRMED anchor, SIM curve]**
- This profile deliberately avoids making roll as heavy as Spitfire-style ailerons. The F4U report describes aileron forces as “reasonably light”; high-speed roll still becomes heavy and clamps beyond the normal maneuvering range. **[CONFIRMED behavior]**

### Rudder / yaw

- `airspeed_spring`: v² curve, **24 kg at 600 km/h** plus 2.2 kg simple spring. This keeps pedals usable at cruise but lets high-speed rudder reach the 14 kg device clamp, matching the report that full rudder could not be obtained at maximum speed. **[CONFIRMED behavior, INFERRED kg]**
- `prop_thrust_spring`: 5 kg peak, rudder only. This makes high-power takeoff/go-around slipstream and torque management tactile without adding a fake aileron force. **[INFERRED]**

## kg force-budget table

Assumptions: values below are full-deflection spring requests only. Shakers/jolts are vibration effects, not kg centering forces. `elevator_weight`, `local_aoa`, and propwash vary with telemetry and are noted separately. Felt force assumes FFBeast Setup **Max force available = 14 kg**.

| Condition | Speed km/h | G used | Elevator req/felt kg | Aileron req/felt kg | Rudder req/felt kg |
|---|---:|---:|---:|---:|---:|
| Taxi / controls check | 0 | 1.0 | 1.0 / 1.0 | 1.6 / 1.6 | 2.2 / 2.2 |
| Carrier approach / stall warning ~82 KIAS | 152 | 1.0 | 1.8 / 1.8 | 2.7 / 2.7 | 3.7 / 3.7 |
| Takeoff / low-speed work ~120 KIAS | 222 | 1.0 | 2.7 / 2.7 | 4.0 / 4.0 | 5.5 / 5.5 |
| Cruise / test anchor 200 KIAS | 370 | 1.0 | 5.8 / 5.8 | 8.3 / 8.3 | 11.3 / 11.3 |
| Maneuvering ~250 KIAS | 463 | 4.0 | 10.0 / 10.0 | 12.1 / 12.1 | 16.5 / 14.0 CLAMP |
| High speed ~350 KIAS | 648 | 1.0 | 15.6 / 14.0 CLAMP | 22.1 / 14.0 CLAMP | 30.2 / 14.0 CLAMP |
| Overspeed test ~700 km/h | 700 | 1.0 | 18.0 / 14.0 CLAMP | 25.5 / 14.0 CLAMP | 34.9 / 14.0 CLAMP |
| High-G corner ~250 KIAS / 6 g | 463 | 6.0 | 11.4 / 11.4 | 12.1 / 12.1 | 16.5 / 14.0 CLAMP |


Propwash can add up to about **+2.5 kg pitch** and **+5.0 kg rudder** at high thrust. Near stall, `local_aoa` reduces pitch airspeed-spring feel and `aoa_shaker` adds mild buffet; exact felt value depends on DCS AoA telemetry. **[SIM]**

## Required manual setup

- FFBeast Setup: set **Max force available = your true measured maximum grip force**, here **14 kg**.
- FFBeast Commander metadata is set to `flight_data_source = 1` and `flight_module = "F4U-1D"`. Verify the exact module string in the Commander telemetry/debug view; if DCS reports a different token, update the JSON metadata.
- Confirm the speed telemetry shown by Commander is IAS/EAS-like and in km/h. Hinge-moment force should scale with IAS/EAS², not TAS.
- DCS fixed-wing DirectX constant force is left at `force = 0`; `trimmer = 100` is retained so DCS trim moves the FFBeast spring neutral.
- Dynamic Motion Range: normally **off for DCS** unless your setup specifically needs it.
- Reduce in-game cockpit/camera shake if it double-counts FFBeast `generic_shaker`, `aoa_shaker`, engine rumble, or gunfire.

## Flight-test checklist

1. **Ground:** with engine off, verify stick returns cleanly, elevator droop does not pull excessively, and taxi rumble is present but not dominant.
2. **Takeoff:** verify rudder gets heavier with power but remains controllable; if too much, reduce rudder `prop_thrust_spring` peak from 5.0 to 3.5 kg.
3. **Approach:** at ~82–90 KIAS, verify pitch is light and stall approach gives subtle lightening/buffet rather than a strong artificial wall.
4. **Cruise 180–220 KIAS:** verify roll full-deflection effort feels around the 7–9 kg range and pitch is lighter than roll for small corrections.
5. **Maneuver 220–270 KIAS:** pull 3–5 g and check that pitch loads up through `g_force_spring` without clamping continuously.
6. **High-speed dive:** verify pitch/rudder/roll stiffen progressively and can clamp near the top end. If the stick feels pinned too early, reduce elevator `airspeed_spring` peak from 12.5 to 10.5 kg or raise its reference speed to 650 km/h.
7. **Weapons:** fire guns and release rockets/bombs; shakers should be present but secondary. Reduce gunfire or weapon jolt by 20–30% if distracting.

## Known uncertainties and tuning recommendations

- Exact F4U-1D elevator stick-force-per-G was not found in a primary numeric table during this pass. Pitch kg values are therefore **[INFERRED]**, intentionally light, and should be flight-tested.
- Rudder kg values are **[INFERRED]** from qualitative primary-source behavior. Reduce rudder airspeed peak from 24 kg to 20 kg if pedals clamp too early.
- F4U-1D aileron force anchors are the strongest data in this profile. If roll feels too heavy at 200 KIAS, reduce only the aileron `simple_spring` from 1.6 to 0.8 kg before changing the v² airspeed curve.
- DCS F4U-1D is Early Access and flight-model/telemetry details may change. Re-test after major module updates.
