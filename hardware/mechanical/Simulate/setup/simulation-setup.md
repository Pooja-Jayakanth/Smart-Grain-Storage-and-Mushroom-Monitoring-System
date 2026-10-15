# Simulation set-up

Everything needed to rebuild and re-run the study in SOLIDWORKS Flow Simulation 2024 SP1.
All values here are **inputs**. Results are in [`../results`](../results).

## 1. Model

| Item | Value |
|---|---|
| Assembly | `Node enclosure_flow.SLDASM` (a copy of `Node enclosure.SLDASM`, configuration `Default`) |
| Units | SI (temperature shown in °C) |
| Coordinate system | Assembly global. **+Y points toward the sensor holder**, so in the hanging position negative Y is up. |
| Components disabled (Component Control) | `Side wall hanger-1` (hidden mounting bracket), `Hanger lock screw envelope-1` (construction body, not a real part) |
| Geometry handling | Mesh Boolean (CAD Boolean failed on this assembly), exclude cavities without flow conditions: on |

## 2. General settings

| Setting | Value |
|---|---|
| Analysis type | External, steady state |
| Physical features | Heat conduction in solids: on · Radiation: on · Gravity: on · Time-dependent: off |
| Gravity vector | X = 0, **Y = +9.81 m/s²**, Z = 0 (device hangs from its tab, sensor holder below the box) |
| Fluid | Air, with humidity |
| Flow type | Laminar and turbulent |
| Default solid | PVC Rigid (only affects parts not covered by the material assignments below) |
| Wall roughness | 0 µm |

## 3. Ambient and initial conditions

Sri Lankan dry-zone paddy store, closed room.

| Parameter | Value |
|---|---|
| Pressure | 101,325 Pa |
| Temperature | 32 °C (305.15 K) |
| Relative humidity | 70 % (reference temperature 32 °C) |
| Velocity | 0 m/s in X, Y, Z (still air) |
| Initial solid temperature | 32 °C |

## 4. Radiation

| Parameter | Value |
|---|---|
| Environment radiation temperature | 32 °C (store walls) |
| Solar radiation | Off (indoors) |
| Radiative surface on all printed parts | **WhitePLA**: emissivity 0.9, solar absorptance 0.25, diffuse |

## 5. Materials

| Parts | Material |
|---|---|
| Main body, buttons closer, button retainer, button plungers, retainer thumb screw, sensor union hand collar, sensor lower hand ring, both sensor holder halves, PCB screws and nuts | **PLA+ white print** (user-defined) |
| `pcb-2` (main PCB with components) | FR4 (Pre-Defined\Laminates) |
| Battery, RH/T sensor board, cables | Default solid |

User-defined material **PLA+ white print**: density 1230 kg/m³, specific heat 1800 J/kg·K, thermal conductivity
0.15 W/m·K (isotropic), dielectric, melting temperature 328.15 K (55 °C, used as the softening warning).
These are typical PLA+ values, not measured on the actual filament.

## 6. Heat sources (volume heat generation, worst case: Wi-Fi always on)

| Source | Bodies in `pcb-2` | Power |
|---|---|---|
| ESP-12F module | `U1~WIRELM…` | 0.50 W |
| Q1 (TO-220) | `Q1~TO-220…` | 0.20 W |
| 8 through-hole resistors | `R2~RES-TH…` (8 bodies) | 0.40 W total |
| LDO regulator (SOT-223) | `U7~SOT-223…` (18 bodies) | 0.25 W total |
| **Total** | | **1.35 W** |

The RH/T sensor board has no heat source. These are estimates; a battery-powered node with deep sleep
dissipates far less.

## 7. Computational domain

| Axis | Min (mm) | Max (mm) | Note |
|---|---|---|---|
| X | −166.9 | 196.8 | automatic |
| Y | −400 | 220 | extended in −Y (above the box) for the rising plume |
| Z | −144.8 | 172.4 | automatic |

## 8. Mesh

| Setting | Value |
|---|---|
| Type | Automatic global mesh |
| Level of initial mesh | 5 |
| Minimum gap size | **1.5 mm** (set manually; the automatic value of 67.6 mm gave only 37,000 cells) |
| Resulting cells | 3,158,364 (fluid 1,680,983 · solid 1,477,381) |

## 9. Goals and convergence

| Goal | Type |
|---|---|
| RH-T sensor avg temp | Volume goal, average solid temperature, component `RH,Tem sensor-1` |
| ESP max temp | Volume goal, maximum solid temperature, ESP-12F body |
| Battery max temp | Volume goal, maximum solid temperature, battery component |

Finish conditions: default (goals converged). The run converged at **382 iterations** (1.6 travels, about 4.7 hours on a
16-thread Intel i5-12500H).

## 10. Probe points

[`probe_points.csv`](probe_points.csv) holds the coordinates in metres (x, y, z), one point per line, for
Results → Point Parameters → Import file.

| # | Location | x, y, z (mm) |
|---|---|---|
| 1 | Sensor chip surface | −5.7, 86, 24 |
| 2 | Sensor holder, centre upper (near neck) | 0, 80, 22.5 |
| 3 | Sensor holder, centre lower | 0, 90, 22.5 |
| 4 | Sensor holder, right | 12, 85, 22.5 |
| 5 | Sensor holder, left | −12, 85, 22.5 |
| 6 | Sensor holder, front | 0, 85, 34.5 |
| 7 | Sensor holder, back | 0, 85, 10.5 |
| 8 | Neck between box and holder | 0, 58, 22.5 |
| 9 | 35 mm below holder | 0, 120, 22.5 |
| 10 | 75 mm below holder | 0, 160, 22.5 |
| 11 | Room air beside box | −70, 0, 12 |
| 12 | Inside box near PCB | −15, 0, 20 |
| 13 | Inside box near neck | −15, 45, 20 |
| 14 | Far ambient | −150, 250, −100 (outside the final domain, no value) |
| 15 | Plume 20 mm above box | 0, −90, 12 |
| 16 | Plume 80 mm above box | 0, −150, 12 |
| 17 | Plume 180 mm above box | 0, −250, 12 |
| 18 | Sensor holder outer edge | 40, 85, 22.5 |

## 11. Result plots

| Image | Plot | Scale |
|---|---|---|
| 01 | Cut plot, Front Plane offset 22.5 mm (through box, neck and holder centre), Temperature | 32–40 °C |
| 02 | Same plane, Relative humidity | 50–70 % |
| 03 | Same plane, Velocity | 0–0.2 m/s |
| 04 | Surface plot, all faces, Temperature (Solid), 3D view | 32–45 °C |

Model geometry was hidden for images 01–03. Views are rotated 180° so the device appears hanging (tab up).
