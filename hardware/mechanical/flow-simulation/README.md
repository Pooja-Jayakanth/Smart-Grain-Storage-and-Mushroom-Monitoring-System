# Enclosure flow simulation: thermal validation of the sensing node

Part of the **Smart Grain Storage and Mushroom Monitoring System** (Team Sisyphus).

**Question:** the node hangs in the store with its temperature and humidity sensor in a louvred holder below the
electronics box. Does the sensor read the real store air, or is it disturbed by heat from the electronics?

**Answer: the enclosure design is validated.** In still air with a worst-case 1.35 W heat load, the sensor reads
**32.12 °C** against 32.00 °C room air (**+0.12 °C**) and **68.9–69.7 % RH** against 70 % (**−0.3 to −1.1 % RH**).
That error is far smaller than the accuracy of the DHT11 used in the final node (about ±2 °C and ±5 % RH, datasheet values).

![Temperature section](results/images/01%20Temperature%20section%20front.png)

*Temperature section through the box, neck and sensor holder (32–40 °C scale). Warm air from the PCB rises out of
the top of the box; the sensor holder below stays at room temperature.*

---

## What was simulated

Tool: SOLIDWORKS Flow Simulation 2024 SP1.

| Parameter | Value |
|---|---|
| Analysis | External, steady state, humid air, laminar and turbulent |
| Store air | 32 °C, 70 % RH, 101.325 kPa (Sri Lankan dry-zone paddy store) |
| Air movement | Still air: closed store, natural convection only (worst case for heat build-up) |
| Gravity | 9.81 m/s², node hanging from its tab, sensor holder below the box |
| Radiation | On; surroundings at 32 °C, no sun; printed surfaces emissivity 0.9 |
| Heat conduction in solids | On |
| Heat load | **1.35 W continuous**, worst case with Wi-Fi always on (see note below) |
| Enclosure material | PLA+ (white): 1230 kg/m³, 0.15 W/m·K, softening limit 55 °C |
| Mesh | 3,158,364 cells, minimum gap 1.5 mm |
| Convergence | 382 iterations, all goals converged |

**Note on the electronics model.** The simulated PCB is the earlier prototype board with an ESP-12F (ESP8266)
Wi-Fi module: ESP-12F 0.50 W, 8 resistors 0.40 W, LDO regulator 0.25 W, MOSFET 0.20 W. The final node uses an
**ESP32**. An ESP32 with Wi-Fi active dissipates a comparable amount, and 1.35 W in total is a deliberate upper bound,
so the conclusions apply to the ESP32 node. A re-run with the final PCB layout is listed under future work.

## Results

| Result | Value | Room air | Difference |
|---|---|---|---|
| Sensor chip temperature | 32.12 °C | 32.00 °C | +0.12 °C |
| Air temperature in sensor holder | 32.08–32.27 °C | 32.00 °C | +0.08 to +0.27 °C |
| Relative humidity in sensor holder | 68.9–69.7 % | 70.0 % | −0.3 to −1.1 % RH |
| Air inside the electronics box | 41.5 °C, 42 % RH | | +9.5 °C |
| Warm plume above the box (20–180 mm) | 35.0–35.7 °C, rising 0.08–0.19 m/s | 32.00 °C | +3.0 to +3.7 °C |
| Hottest PLA+ part | about 50.4 °C | limit 55 °C | 4.6 °C margin |
| Wi-Fi module hottest point | 82.5 °C | | |

Heat leaves the node roughly half by convection (about 0.70 W) and half by radiation (about 0.65 W).

| Relative humidity | Air speed | Surface temperature |
|---|---|---|
| ![RH](results/images/02%20Relative%20humidity%20section%20front.png) | ![Speed](results/images/03%20Air%20speed%20section%20front.png) | ![Surface](results/images/04%20Surface%20temperature%203D.png) |

All 17 probe points are in [`results/Probe point results.csv`](results/Probe%20point%20results.csv); the full
write-up is in [`results/Results summary.txt`](results/Results%20summary.txt).

## Why the sensor goes below the box

| Sensor position | Sensor temperature | Sensor RH | Result |
|---|---|---|---|
| Above the box | about 49 °C | 26–31 % | Not usable |
| **Below the box (final design)** | **32.12 °C** | **68.9–69.7 %** | Accurate |

With the sensor above, warm air from the electronics flows straight into the sensor holder. The "above" case comes
from an earlier run with PVC parts and radiation off, so it shows the trend rather than an exact comparison.

## Design rules for deployment

* Hang the node tab up, sensor holder down. Do not mount it sideways or upside down.
* Do not place another node or anything sensitive directly above a node; the plume is 3–4 °C warmer and drier.
* The sensor holder follows room changes over minutes, which suits grain storage, where conditions change over hours.
* PLA+ keeps a 4.6 °C margin at 32 °C and worst-case power. For stores reaching 38–40 °C, print in PETG or ASA,
  or reduce power with ESP32 deep sleep.

## Limitations

* No store walls, grain bags or shelving were modelled; the node hangs in open still air.
* The sensor measures the air around the node, not the air inside the grain bulk.
* The 1.35 W heat load is a worst-case estimate; with deep sleep the real heat, and the sensor error, will be lower.
* Hot spots inside the PCB (up to about 126 °C) are not reliable, because components were modelled as solid FR4.
  They do not affect the airflow or sensor results.
* Mushroom-house conditions (higher humidity, lower temperature) were not simulated.

## Folder contents

```
results/   Results summary, probe point CSV and result images
report/    LaTeX source of the simulation report
setup/     Full simulation set-up and probe point coordinates (for Point Parameters import)
```

The solver output files (hundreds of MB) are not included; they are regenerated by running the project.
The SOLIDWORKS assembly and Flow Simulation project are available from the team on request.
