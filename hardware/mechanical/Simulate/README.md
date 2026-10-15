# Sisyphus Paddy-Store Sensor Node: Flow Simulation

Airflow and thermal feasibility study of the **Sisyphus IoT sensor node**, a wall-hung enclosure that monitors
air temperature and relative humidity in paddy storage rooms in the dry zone of Sri Lanka.

**Question:** does the temperature/humidity sensor, inside the louvred sensor holder, read the real store air,
or is it disturbed by the heat from the electronics box?

**Answer: feasible.** With the device hanging from its tab and the sensor holder **below** the electronics box,
the warm air from the PCB rises up and away. In still air with a worst-case 1.35 W heat load, the sensor reads
**32.12 °C** against 32.00 °C room air, and **68.9–69.7 % RH** against 70 %. That error is smaller than the accuracy
of a typical RH/T sensor (about ±0.3 °C, ±2–3 % RH).

![Temperature section](results/images/01%20Temperature%20section%20front.png)

*Temperature section through the box, neck and sensor holder (32–40 °C scale). Hot air inside the box rises out
of the top; the sensor holder below stays at room temperature.*

---

## Device

| Part | Description |
|---|---|
| Electronics box | 3D-printed PLA+ enclosure with side vent slots; ESP-12F (ESP8266) PCB, regulator, battery |
| Sensor holder | Louvred radiation shield below the box, holding the RH/temperature sensor board |
| Mounting | Hangs from a tab at the top; sensor holder points down |
| Overall size | about 82 × 86 × 141 mm |

## Simulation set-up

Tool: SOLIDWORKS Flow Simulation 2024 SP1.

| Parameter | Value |
|---|---|
| Analysis type | External, steady state, humid air, laminar and turbulent |
| Room air | 32 °C, 70 % RH, 101.325 kPa |
| Air movement | Still air (closed store, natural convection only) |
| Gravity | 9.81 m/s², device hanging, sensor holder below the box |
| Radiation | On; surroundings at 32 °C, no sun; printed surfaces emissivity 0.9 |
| Heat conduction in solids | On |
| PCB heat (worst case, Wi-Fi always on) | ESP-12F 0.50 W, 8 resistors 0.40 W, LDO regulator 0.25 W, Q1 0.20 W; **1.35 W total** |
| Sensor board heat | 0 W |
| Enclosure material | PLA+ (white): 1230 kg/m³, 0.15 W/m·K, 1800 J/kg·K, softening limit 55 °C |
| PCB material | FR4 |
| Mesh | Level 5, minimum gap 1.5 mm, 3,158,364 cells |
| Convergence | 382 iterations, all goals converged |

## Results

| Result | Value | Room air | Difference |
|---|---|---|---|
| Sensor chip temperature | 32.12 °C | 32.00 °C | +0.12 °C |
| Air temperature in sensor holder | 32.08–32.27 °C | 32.00 °C | +0.08 to +0.27 °C |
| Relative humidity in sensor holder | 68.9–69.7 % | 70.0 % | −0.3 to −1.1 % RH |
| Air inside box (near PCB) | 41.5 °C, 42 % RH | | +9.5 °C |
| Warm plume above the box (20–180 mm) | 35.0–35.7 °C, rising 0.08–0.19 m/s | 32.00 °C | +3.0 to +3.7 °C |
| Hottest PLA+ part | ≈ 50.4 °C | limit 55 °C | 4.6 °C margin |
| ESP-12F hottest point | 82.5 °C | | |

Heat leaves the device roughly half by convection (≈ 0.70 W) and half by radiation (≈ 0.65 W).

All 17 probe points are in [`results/Probe point results.csv`](results/Probe%20point%20results.csv), and the full write-up is in
[`results/Results summary.txt`](results/Results%20summary.txt).

| Relative humidity | Air speed | Surface temperature |
|---|---|---|
| ![RH](results/images/02%20Relative%20humidity%20section%20front.png) | ![Speed](results/images/03%20Air%20speed%20section%20front.png) | ![Surface](results/images/04%20Surface%20temperature%203D.png) |

## Why the sensor goes below the box

| Sensor position | Sensor temperature | Sensor RH | Result |
|---|---|---|---|
| Above the box | ≈ 49 °C | 26–31 % | Not usable |
| **Below the box (final design)** | **32.12 °C** | **68.9–69.7 %** | Accurate |

With the sensor above, the warm air from the electronics flows up through the neck into the sensor holder.
The "above" case came from an earlier run with PVC parts and radiation off, so it shows the trend rather than
an exact like-for-like comparison.

## Design notes

* Keep the hanging orientation: tab up, sensor holder down.
* Do not mount anything sensitive directly above the device; the plume is 3–4 °C warmer and up to 13 % RH drier.
* Air inside the sensor holder moves slowly in still air (about 0.001–0.014 m/s), so readings follow room changes over minutes. This is adequate for paddy storage, which changes over hours.
* PLA+ has a 4.6 °C margin at 32 °C with worst-case power. On hotter days (38–40 °C) that margin is used up; PETG or ASA, or ESP deep sleep, is safer.

## Limitations

* No store wall or paddy bags were modelled; the device hangs in open still air.
* The sensor measures air near the device, not the air inside the paddy bags.
* The 1.35 W heat load is a worst-case estimate. With ESP deep sleep the real heat, and the sensor error, will be lower.
* Hot spots inside the PCB (up to about 126 °C) are not reliable, because whole components were modelled as FR4. They do not affect the airflow or sensor results.
* Probe values were transcribed from the SOLIDWORKS results table.
* The hottest PLA+ temperature is calculated from SOLIDWORKS's "overheat above melting temperature" value (55 − 4.56 °C).
* The heat-balance split comes from solver goal values.

## Repository contents

```
results/   Results summary, probe point CSV, result images
report/    LaTeX source and PDF of the results
setup/     Simulation inputs and probe point coordinates
model/     SOLIDWORKS assembly (Pack and Go) to reproduce the study
```

The solver's raw output files (hundreds of MB) are not included. Open the assembly in SOLIDWORKS Flow Simulation
and run the project to regenerate them.

## Reproduce

1. Unzip `model/` and open `Node enclosure_flow.SLDASM` in SOLIDWORKS 2024 with Flow Simulation enabled.
2. Add the material **PLA+ white print** and the radiative surface **WhitePLA** (emissivity 0.9) to the Engineering Database using the values above.
3. Run the project `Paddy store still air 32C 70RH`; about 4–5 hours on a 16-thread laptop.
4. Results → Point Parameters → Import `setup/probe_points.csv` to get the probe table.
