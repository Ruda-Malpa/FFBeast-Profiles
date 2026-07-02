# FFBeast Commander profile notes — IL-2 Korea MiG-15bis

Profile file: `IL2_Korea_MiG-15bis_FFBeast_profile.json`  
Target simulator/module: **IL-2 Korea — MiG-15bis**  
Reference rig: FFBeast base, ~15 cm stick extension, **14 kg true maximum grip force**.

## Evidence summary

- **Control system** — [CONFIRMED]: the MiG-15bis is a single-seat swept-wing VK-1 fighter; the ailerons have a hydraulic booster with about **2.7:1** boost advantage, while elevator and rudder are manually controlled. The manual also describes electric elevator and aileron trim tabs.
- **Pilot handling cross-check** — [CONFIRMED]: a modern MiG-15 pilot report describes medium elevator forces on the ground, stiff ailerons with boost off, good boosted aileron feel, quite light rudder, good stall airframe buffet, pitch sensitivity at 300–400 kt, and possible high-speed stick-freeze/compressibility behavior.
- **IL-2 Korea status** — [CONFIRMED/SIM]: IL-2 Korea includes the flyable MiG-15bis; profile module matching should be verified in Commander because the exact live telemetry module string may differ.
- **Real force numbers** — [UNKNOWN]: I did **not** find a reliable MiG-15bis stick-force-per-g or force-vs-speed chart. The kg magnitudes below are conservative engineering inferences from control-system type and pilot reports, not confirmed flight-test numbers.

## Aircraft / control-system classification

| Axis | Real system | Profile primary force | Evidence tag |
|---|---|---|---|
| Elevator | Manual/aerodynamically loaded, electric trim tab | `airspeed_spring` v² + low `simple_spring`; small `g_force_spring` wall/cue | [CONFIRMED] system, [INFERRED] kg |
| Ailerons | Hydraulically boosted, 2.7:1 boost advantage; can be stiff boost-off | modest `airspeed_spring` + low `simple_spring` | [CONFIRMED] system, [INFERRED] kg |
| Rudder | Manual, but pilot report says quite light | light `airspeed_spring` + low `simple_spring` | [CONFIRMED] system/handling, [INFERRED] kg |

## Force model

FFBeast Commander spring outputs are authored in real kg at `total_effect_strength = 100`. Requested force is:

`requested_kg = curve_out × total_effect_strength / 100`

On this rig, felt force is:

`felt_kg = min(requested_kg, 14)`

This profile uses **literal-kg authoring** for normal flight, with only the elevator high-speed/high-G corner intentionally clamping. All spring strengths below are therefore absolute kg requests, not percentages of device maximum.

## Main force settings

| Axis | Effect | Peak / shape | Rationale | Tag |
|---|---:|---:|---|---|
| Elevator | `airspeed_spring` | 18 kg at 900 km/h, v² | Manual elevator should load with dynamic pressure; high-speed stick-freeze caution justifies a strong high-speed wall without making cruise unusable. | [INFERRED] |
| Elevator | `simple_spring` | 2.0 kg at full deflection | Provides low-speed/ground centering and a small breakout without elevator droop; pilot report says stick stays where placed on ground. | [INFERRED] |
| Elevator | `g_force_spring` | 0–9 kg from 0–10 g, 5 kg at 8 g | Structural/G awareness cue. Real MiG-15 SFpG unknown, so this is deliberately modest below 6 g and only becomes strong near the limit. | [SIM]/[INFERRED] |
| Ailerons | `airspeed_spring` | 8 kg at 900 km/h, v² | Ailerons are hydraulically boosted, so roll feel should not become warbird-heavy; still not feather-light. | [INFERRED] |
| Ailerons | `simple_spring` | 1.6 kg full deflection | Boosted/artificial residual feel and breakout. | [INFERRED] |
| Rudder | `airspeed_spring` | 10 kg at 850 km/h, v² | Rudder is manual but reported quite light; gives speed loading without early clamp. | [INFERRED] |
| Rudder | `simple_spring` | 1.2 kg full pedal | Pedal centering/friction cue. | [INFERRED] |

## Requested / felt kg budget

Assumptions: full control displacement for each spring at the named IAS; real in-flight stick force scales with actual displacement around trim. Values are `requested / felt` kg on a 14 kg rig.

| Condition | IAS km/h | G | Elevator kg | Aileron kg | Rudder kg | Notes |
|---|---:|---:|---:|---:|---:|---|
| Taxi / stopped | 0 | 1 | 0.00 / 0.00 | 0.00 / 0.00 | 0.00 / 0.00 | neutral controls; simple spring only when displaced |
| Takeoff rotation | 227 | 1 | 3.15 / 3.15 | 2.11 / 2.11 | 1.91 / 1.91 | pilot manual: aircraft leaves ground about 227 km/h |
| Final approach | 250 | 1 | 3.39 / 3.39 | 2.22 / 2.22 | 2.07 / 2.07 | pilot manual: final about 250 km/h |
| Low-altitude cruise | 600 | 1 | 10.00 / 10.00 | 5.16 / 5.16 | 6.18 / 6.18 | pilot manual cruise band near sea level 565-620 km/h |
| Typical medium-alt cruise | 450 | 1 | 6.50 / 6.50 | 3.60 / 3.60 | 4.00 / 4.00 | manual cruise bands around 435-485 km/h depending altitude |
| 400 kt fast turn, 2 g | 740 | 2 | 14.37 / 14.00 | 7.01 / 7.01 | 8.78 / 8.78 | pilot report: 400 kt / 60° bank / 2 g reference |
| High-speed / overspeed cue | 900 | 1 | 20.00 / 14.00 | 9.60 / 9.60 | 12.41 / 12.41 | compressibility/stick-freeze caution region |
| Structural warning, 650 km/h / 8 g | 650 | 8 | 16.39 / 14.00 | 5.77 / 5.77 | 7.05 / 7.05 | sim safety cue; wall intentionally clamps on pitch |

## Shakers and event cues

- `aoa_shaker`: enabled on elevator and ailerons for stall/buffet onset. This is a shaker cue, not a centering-force change. [SIM]
- `generic_shaker`: kept active at modest strength for IL-2 telemetry vibration/buffet. [SIM]
- `over_g_shaker`: starts gently above ~5.5 g and becomes strong near 8–9 g. [SIM]
- `airbrake_shaker`: moderate, because MiG-15 speed brakes are important and high-speed deployment should be noticeable. [SIM]
- `flaps_shaker`/`gear_shaker`: light to moderate mechanization buffet only; they do not change centering force. [SIM]
- `gunfire_shaker`: three gun channels are active to represent the 37 mm + 2×23 mm armament if IL-2 telemetry maps gun channels separately. [SIM]
- No afterburner or countermeasure effects are included; MiG-15bis has neither. [CONFIRMED/SIM]

## Required manual setup

1. **FFBeast Setup:** set **Max force available = true measured maximum**, here **14 kg**. Do not use this as a volume knob.
2. **FFBeast Commander / IL-2:** configure IL-2 telemetry in Commander, update IL-2 `startup.cfg`, then restart Commander if required.
3. **Profile match:** this JSON uses `flight_module = "MiG-15bis"`. After the first IL-2 Korea flight, copy the exact module string shown in Commander’s device/telemetry state and update the profile if the live string differs.
4. **Flight data source:** this JSON uses `flight_data_source = 0` as an import placeholder. In Commander, select/save it under the IL-2 telemetry source so the numeric source matches your setup.
5. **Dynamic Motion Range:** enable it for IL-2. IL-2 scales available stick movement with speed; DMR compensates so the physical FFBeast range remains usable.
6. **Telemetry units:** verify in Commander telemetry/debug that the speed driving `airspeed_spring` is IAS/EAS-like, not TAS. If it is TAS, high-altitude forces will be too heavy and the speed springs should be retuned.
7. Reduce any in-game or external shake that double-counts FFBeast shakers.

## Flight-test checklist

1. **Ground:** stick should not droop forward; elevator should feel medium-light with small breakout. If it recenters too aggressively at zero speed, reduce elevator `simple_spring` peak from 2.0 to 1.5 kg.
2. **Takeoff:** rotation around ~227 km/h should be easy; if pitch feels too heavy, reduce elevator `airspeed_spring` peak 18 → 15 kg.
3. **Cruise 450–600 km/h:** elevator should be clearly heavier than roll, but not a wall. Roll should feel boosted and smooth, not warbird-stiff.
4. **400 kt / ~740 km/h fast turn:** pitch should approach the 14 kg clamp if you pull hard; that is intentional. If the aircraft becomes un-flyable, reduce `g_force_spring` 8 g point from 5 kg to 3.5 kg.
5. **Stall:** there should be strong buffet cue before departure; if IL-2 already provides heavy generic buffet, reduce `aoa_shaker.total_effect_strength` by 25–40%.
6. **Speed brake:** deploy at speed and verify a noticeable but not violent vibration.
7. **G-limit:** in a test mission, determine where IL-2 Korea actually damages the MiG-15bis; shift `over_g_shaker` onset to start about 1 g below that practical break point.

## Known uncertainties

- [UNKNOWN] Real MiG-15bis stick-force-per-g and exact force-vs-IAS curves.
- [UNKNOWN] Exact IL-2 Korea telemetry module string and `flight_data_source` numeric ID on your installation.
- [SIM] Exact IL-2 Korea airframe overstress threshold may differ from published/other-sim MiG-15 limits; over-G cues should be flight-tested.
- [SIM] IL-2 generic buffet strength can vary by aircraft/update; tune `generic_shaker` and `aoa_shaker` after one stall series.

## Source pointers used

- MiG-15bis operating manual via GovernmentAttic / NASIC archive: control-system layout, speeds, trim, takeoff/landing/cruise references.
- IAOPA / General Aviation pilot report: qualitative handling, aileron boost feel, rudder lightness, stall buffet, high-speed stick-freeze caution.
- FFBeast Commander and Setup documentation: kg semantics, Max Force Available, IL-2 telemetry setup, Dynamic Motion Range, generic shaker/smart damper semantics.
- Official IL-2 Korea page and current early-access coverage: MiG-15bis module context and need to verify live module string.
