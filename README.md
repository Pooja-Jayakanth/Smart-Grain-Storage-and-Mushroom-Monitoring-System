# Smart Grain and Mushroom Monitoring System

## IoTrix 2.0 – Track C
### IoT Product & System Architecture

## Project Overview

The **Smart Grain and Mushroom Monitoring System** is a low-cost distributed IoT platform designed to monitor environmental conditions in **grain-storage facilities and mushroom-growing environments**.

Both applications are highly sensitive to environmental conditions. Temperature and relative humidity can significantly affect grain quality, fungal activity, mushroom growth conditions, and overall storage or cultivation performance.

The system uses multiple smart sensing nodes placed at selected locations within the monitored environment. Each node measures important environmental parameters, performs local processing, and transmits live telemetry to a centralized dashboard.

The project is being developed in three stages:

1. **Semi-final proof-of-concept prototype**
2. **Final competition product-level sensing node**
3. **Future data-driven, CO₂-monitoring, and machine-learning implementation**

---

# Problem Statement

Environmental conditions can vary significantly at different locations within a grain-storage warehouse or mushroom-growing facility.

For grain storage, unfavorable combinations of temperature and humidity can create conditions associated with spoilage, fungal activity, and deterioration.

For mushroom cultivation, temperature and relative humidity must remain within suitable ranges to maintain appropriate growing conditions.

Periodic manual measurements or a single sensing point may not provide sufficient information about spatial variations across the monitored environment.

A distributed monitoring system is therefore needed to:

- Continuously monitor environmental conditions at multiple locations
- Identify abnormal or unfavorable conditions early
- Provide centralized real-time visualization
- Generate local and remote alerts
- Store data for future analysis and predictive modelling

---

# Proposed Solution

The proposed system consists of multiple **ESP32-based smart environmental sensing nodes** deployed at selected locations.

Each node acquires environmental measurements, performs local processing, and transmits the processed data through **Wi-Fi using MQTT** to a centralized **Node-RED dashboard**.

The final system is designed to monitor:

- Temperature
- Relative humidity
- Node connectivity and operating status
- Overall environmental-condition level

The measured parameters will be combined using threshold-based decision logic derived from relevant research papers, technical sources, grain-storage guidance, and mushroom-growing requirements.

The dashboard will provide an interpreted condition such as:

- **NORMAL**
- **WARNING**
- **CRITICAL**

If unfavorable conditions are detected:

- A warning will be displayed on the dashboard
- A local buzzer at the relevant sensing node will be activated

This provides both **remote monitoring** and **local warning capability**.

---

# Target Applications

## Grain Storage Monitoring

The system can be deployed at multiple locations inside grain-storage facilities to monitor conditions associated with grain deterioration and fungal-growth risk.

The main parameters are:

- Fused temperature
- Relative humidity

The system is intended to indicate whether environmental conditions are becoming unfavorable rather than directly claiming the presence of fungal growth.

## Mushroom Environment Monitoring

The same distributed architecture can also be used in mushroom-growing environments.

The sensing nodes can continuously monitor:

- Temperature
- Relative humidity

The dashboard can indicate whether the environmental conditions remain within the selected operating ranges for the target mushroom-growing stage.

The threshold values for grain storage and mushroom monitoring will be application-specific and supported by relevant research and technical references.

---

# Development Stages

## 1. Semi-Final Prototype

The prototype demonstrated at the IoTrix 2.0 semi-final is based on the previously developed distributed temperature-monitoring platform.

The current prototype consists of **two operational wireless sensing nodes** connected to a centralized Node-RED monitoring dashboard.

### Components Used in the Semi-Final Prototype

Each prototype node currently contains:

- ESP32 microcontroller
- DB18S20 temperature sensor
- NTC thermistor
- NEO-6M GPS module
- OLED display
- Power-management circuitry
- Push button

For temperature measurement, a sensor-fusion architecture is implemented.

Temperature measurements from:

- DB18S20
- NTC thermistor

are processed using the:

- Calibration
- Sensor validation
- Outlier rejection
- Weighted Least Squares sensor fusion
- Kalman filtering


---

# Temperature Processing Architecture

```text
DB18S20 Temperature  NTC Temperature
        │               |
        │               |
        |               |
        │               │
        ▼               ▼
    Calibration     Calibration
        │               |
        │               |
        │               │
        └───────┬───────┘
                ▼
        Validity Checking
                ▼
         Outlier Rejection
                ▼
      Weighted Least Squares
          Sensor Fusion
                ▼
         Kalman Filtering
                ▼
       Final Temperature
```

Weighted Least Squares is used because the two temperature sensors have different measurement uncertainties.

Instead of assigning equal importance to both measurements, the fusion algorithm gives greater contribution to the more reliable sensor according to experimentally determined characteristics. Weights are given as inverse variances.

The fused output is then passed through a Kalman filter to reduce short-term fluctuations.

---

# Relative Humidity Measurement

The DHT11 also provides relative-humidity measurements.

Relative humidity is important in both target applications.

### For Grain Storage

Relative humidity contributes to the overall assessment of storage conditions associated with moisture-related deterioration and fungal-growth risk.

### For Mushroom Monitoring

Relative humidity is a key environmental parameter for maintaining suitable mushroom-growing conditions.

The measured RH value will therefore be evaluated together with the fused temperature in the final condition-assessment logic.

---

# Purpose of the GPS Module in the Prototype

The semi-final prototype includes a **NEO-6M GPS module** mainly for:

- Prototype development
- Location verification
- Node-position identification
- Testing
- Demonstration purposes

The GPS is not intended to operate continuously in the final product.

In many real deployments, sensing nodes will remain at known fixed positions. Keeping a GPS module active continuously would therefore add unnecessary hardware operation and power consumption.

---

# GPS Strategy in the Final Product

The custom PCB includes the required connections for temporarily connecting the GPS module.

During deployment, the GPS module can be connected when outdoor position acquisition is required.

The acquired location can then be saved by the system.

After the location has been stored, the GPS module can be removed.

```text
Installation
     ↓
Connect GPS Module
     ↓
Acquire Node Location
     ↓
Save Location Data
     ↓
Remove GPS Module
     ↓
Normal Monitoring Operation
```

This provides location capability when required without requiring a permanent GPS module in every deployed node.

---

# Purpose of the OLED in the Prototype

The OLED display is included in the semi-final prototype mainly for:

- Development
- Debugging
- Sensor verification
- Checking node status
- Demonstrating live operation

The OLED allows the development team to directly observe sensor readings and node status during testing.

---

# OLED Removal in the Final Product

The OLED is not required in the intended final monitoring node.

In practical deployment, the centralized dashboard will be the main user interface.

Removing the OLED from the final node helps to:

- Reduce unnecessary hardware
- Reduce power consumption
- Reduce product cost
- Simplify the sensing node
- Improve suitability for distributed deployment

---

# Final Competition Node Architecture

The final competition version of the sensing node is planned to include:

- ESP32 microcontroller
- DHT11 temperature and relative-humidity sensor
- NTC thermistor
- Buzzer
- Custom PCB
- Power-management circuitry
- Temporary GPS connection capability
- Local configuration / storage capability
- Custom mechanical enclosure

The GPS and OLED used in the development prototype will therefore not remain as continuously operating components in the final deployed node.

---

# Environmental Condition Assessment

The final system evaluates environmental conditions using:

- Fused temperature
- Relative humidity

The condition-assessment thresholds will be selected from relevant research papers and suitable technical sources.

Different threshold sets can be used depending on the selected application:

### Grain Storage Mode

The system will assess whether the environmental conditions are:

- Suitable
- Approaching an unfavorable range
- Associated with increased deterioration / fungal-growth risk

### Mushroom Monitoring Mode

The system will assess whether the environmental conditions are:

- Within the desired cultivation range
- Approaching an unsuitable range
- Outside the selected environmental limits

The system does not treat one sensor reading as direct proof of spoilage, contamination, fungal growth, or crop failure. Instead, it provides an **environmental-risk or suitability indication**.

---

# Condition Levels

```text
NORMAL
   │
   ├── Temperature and RH within selected operating limits
   │
WARNING
   │
   ├── One or more parameters approaching an unfavorable range
   │
CRITICAL
   │
   └── Temperature and/or RH outside selected acceptable limits
```

The exact limits will be documented together with the supporting research references.

---

# Live Telemetry

The system operates as a live IoT telemetry network.

Each sensing node continuously performs:

```text
Environmental Sensing
        ↓
Edge Processing
        ↓
Parameter Combination
        ↓
Condition Evaluation
        ↓
Wi-Fi
        ↓
MQTT
        ↓
Centralized Dashboard
```

Each node has a unique Node ID so that measurements from different locations can be identified independently.

---

# Communication Architecture

The sensing nodes communicate using:

- Wi-Fi
- MQTT
- Structured telemetry messages

```text
Sensing Node 01 ─┐
                 │
Sensing Node 02 ─┼── Wi-Fi
                 │
Sensing Node N ──┘
        ↓
   MQTT Broker
        ↓
     Node-RED
        ↓
Central Dashboard
```

MQTT provides a lightweight publish/subscribe communication method suitable for transmitting measurements from multiple IoT sensing nodes.

---

# Dashboard

The centralized monitoring dashboard is implemented using **Node-RED**.

The final dashboard is intended to display:

- Node ID
- Fused temperature
- Relative humidity
- Node connectivity
- Selected monitoring mode
- Environmental-condition classification
- Warning / critical status
- Historical trends
- Live telemetry
- Node location information
- Alarm indication

The dashboard therefore presents both the measured environmental parameters and the interpreted condition.

---

# Alert System

The final system contains both remote and local alert mechanisms.

## Dashboard Alert

When unfavorable environmental conditions are detected:

- The node status changes
- A warning or critical indication appears
- The affected sensing node is identified

## Local Buzzer Alert

A buzzer installed in the sensing node provides a local warning.

```text
Unfavorable Condition Detected
             ↓
      Decision Algorithm
          ┌──┴──┐
          ↓     ↓
     Dashboard  Buzzer
       Alert    Alert
```

This allows the system to provide an alert both remotely and directly at the monitored location.

---

# Prototype vs Final Product

| Feature | Semi-Final Prototype | Final Competition Model |
|---|---|---|
| ESP32 | ✅ | ✅ |
| DHT11 | ❌ | ✅ |
| DB18S20 | ✅ | ❌ Replaced by DHT11 |
| NTC Thermistor | ✅ | ✅ |
| Temperature Sensor Fusion | ✅ | ✅ |
| Relative Humidity | ✅ | ✅ |
| IRLZ44N | ✅ | ✅ |
| Wi-Fi | ✅ | ✅ |
| MQTT | ✅ | ✅ |
| Node-RED Dashboard | ✅ | ✅ |
| GPS | ✅ Development / verification | Temporary connection only |
| OLED | ✅ Development / verification | ❌ Removed |
| Push button | ✅ | ❌ Removed |
| Buzzer | Under development | ✅ |
| Custom PCB | Designed | ✅ |
| Custom Enclosure | Designed | ✅ |
| Live Telemetry | ✅ | ✅ |
| Grain Condition Assessment | Under development | ✅ |
| Mushroom Condition Assessment | Under development | ✅ |
| CO₂ Monitoring | ❌ | Future implementation |
| Machine Learning | ❌ | Future development |

---

# Current Proof of Concept

At the semi-final stage, the project has achieved:

- Two operational sensing nodes
- DB18S20 temperature measurement
- NTC temperature measurement
- Temperature sensor calibration
- WLS temperature sensor fusion
- Kalman filtering
- Sensor validity checking
- ESP32 edge processing
- Wi-Fi connectivity
- MQTT telemetry
- Node identification
- Centralized Node-RED dashboard
- GPS-based prototype location verification
- OLED-based local prototype visualization
- Custom PCB design
- Mechanical enclosure design

This demonstrates the complete core IoT chain:

```text
Physical Measurement
        ↓
Edge Processing
        ↓
Wireless Communication
        ↓
IoT Messaging
        ↓
Centralized Monitoring
```

---

# Technology Stack

## Hardware

- ESP32
- DHT11 temperature and relative-humidity sensor
- DB18S20 for prototype development
- NTC thermistor
- Buzzer
- NEO-6M GPS for development / temporary location initialization
- OLED for prototype development
- Pushbutton for prototype development
- Custom PCB
- Custom enclosure
- Power-management circuitry
- IRLZ44N MOSFET

## Embedded Processing

- Sensor acquisition
- Calibration
- Validity checking
- Outlier rejection
- Weighted Least Squares sensor fusion
- Kalman filtering
- Temperature and RH condition evaluation
- Alarm control

## Communication

- Wi-Fi
- MQTT
- Structured telemetry payloads

## Monitoring

- Node-RED
- Real-time dashboard
- Multi-node visualization
- Historical trends
- Alerts
- Telemetry logging

---

# Engineering Design Philosophy

## Low Cost

The sensing nodes use low-cost and widely available embedded components so that multiple nodes can be deployed practically.

## Distributed Monitoring

Multiple sensing nodes can be placed at different locations instead of relying on a single measurement point.

## Edge Processing

Important sensor processing is performed directly on the ESP32 before data transmission.

## Modular Architecture

The same node and communication architecture can support different environmental-monitoring applications.

## Reduced Final Hardware

Development-only components such as the OLED and continuously operating GPS are removed from the final product when no longer required.

## Dual-Parameter Monitoring

The current condition assessment uses temperature and relative humidity, which are directly relevant to the environmental suitability of grain storage and mushroom cultivation.

## Dual-Application Architecture

The same sensing platform can support:

- Grain-storage environmental monitoring
- Mushroom-growing environmental monitoring

The monitoring logic and threshold values can be configured according to the selected application.

---

# Current Limitations

The current system is still at the prototype and semi-final development stage.

Current limitations include:

- Buzzer integration is still being completed
- DHT11 calibration and integration is still being completed.
- Grain-condition assessment logic requires final validation
- Mushroom-condition assessment logic requires final validation
- Threshold values must be fully documented using relevant research sources
- The custom PCB requires physical fabrication and validation
- Long-duration testing in representative environments is still required
- Power consumption and battery life require further characterization
- Wi-Fi availability may affect live communication
- Larger-scale multi-node deployment requires further validation

---

# Planned Improvements Before the Final Competition

The following improvements are planned before the final competition:

1. Complete buzzer integration.
2. Finalize the combined temperature and RH assessment logic.
4. Define separate threshold sets for grain storage and mushroom monitoring.
5. Document threshold values using relevant research papers and technical sources.
6. Manufacture and test the custom PCB.
7. Integrate the PCB into the designed enclosure.
8. Remove the OLED from the final node.
9. Implement temporary GPS-based location initialization.
10. Improve the dashboard for multi-parameter monitoring.
11. Add application selection for grain-storage or mushroom-monitoring mode.
12. Implement dashboard and local buzzer alerts.
13. Perform longer-duration sensor and communication testing.
14. Test the system under representative grain-storage and mushroom-growing conditions.
15. Measure power consumption and estimate operating duration.
16. Improve Wi-Fi reconnection and network-loss handling.

---

# Future Implementation – CO₂ Level Monitoring

A future version of the system may include a **dedicated NDIR CO₂ sensor** to provide direct CO₂ concentration measurements in ppm.

CO₂ monitoring could add useful information for both applications.

## Grain Storage

Changes in CO₂ concentration can provide an additional indicator of biological respiration and changing storage conditions. In a future implementation, CO₂ measurements could be evaluated together with fused temperature, relative humidity, and historical trends to strengthen environmental-risk assessment.

## Mushroom Monitoring

CO₂ concentration is an important environmental parameter in mushroom-growing environments. A future node could therefore use direct NDIR CO₂ monitoring together with temperature and relative humidity to provide more complete cultivation-condition monitoring.

The future sensing concept would become:

```text
Temperature
    +
Relative Humidity
    +
NDIR CO₂ Measurement
    ↓
Improved Environmental
Condition Assessment
```

A dedicated NDIR sensor would be used rather than estimating CO₂ indirectly.

---

# Future Development – Data Logging and Machine Learning

The current system uses research-based thresholds and interpretable engineering decision logic.

As a future development, long-term environmental data collected from deployed sensing nodes will be stored and used to develop application-specific datasets.

The datasets may include:

- Temperature
- Relative humidity
- CO₂ concentration when future CO₂ sensing is implemented
- Time information
- Node / location information
- Selected application
- Environmental-condition classification
- Verified grain-quality observations where reliable labels are available
- Verified mushroom-growth or environmental observations where reliable labels are available

These datasets can later be used to investigate machine-learning methods for more advanced prediction.

## Future Grain Monitoring

A validated model could be investigated for earlier identification of environmental patterns associated with grain deterioration risk.

## Future Mushroom Monitoring

A separate validated model could be investigated for identifying environmental patterns associated with suitable or unsuitable mushroom-growing conditions.

The long-term development path is:

```text
Live IoT Monitoring
        ↓
Historical Data Collection
        ↓
Dataset Development
        ↓
CO₂ Monitoring Extension
        ↓
Data Analysis
        ↓
Machine-Learning Model Training
        ↓
Model Validation
        ↓
Deployment of Validated Model
        ↓
Improved Predictive Monitoring
```

Machine learning is considered a future enhancement rather than a requirement for the current proof of concept.

The current threshold-based system provides an interpretable engineering baseline before sufficient representative and correctly labelled field data are available for model training.

---

# Expected Final System Architecture

```text
       SMART GRAIN & MUSHROOM
          MONITORING SYSTEM

 ┌─────────────────────────────────────┐
 │               NODE 01               │
 │                                     │
 │ DHT11 → Temperature + RH            │
 │ NTC   → Temperature                 │
 │                                     │
 │                                     │
 │ ESP32                               │
 │ ├── Calibration                     │
 │ ├── Sensor Validation               │
 │ ├── Temperature Sensor Fusion       │
 │ ├── Kalman Filtering                │
 │ ├── Condition Assessment            │
 │ └── Alarm Logic                     │
 │                                     │
 │ Buzzer                              │
 └──────────────────┬──────────────────┘
                    │
                    │ Wi-Fi / MQTT
                    │
        ┌───────────┴───────────┐
        │                       │
      NODE 02                 NODE N
        │                       │
        └───────────┬───────────┘
                    ▼
               MQTT Broker
                    ▼
                 Node-RED
                    ▼
            CENTRAL DASHBOARD
                    │
       ┌────────────┼────────────┐
       ▼                         ▼
 Temperature                     RH
       │                         │
       └────────────┼────────────┘
                    ▼
       Application-Specific
        Condition Assessment
                    ▼
         NORMAL / WARNING /
              CRITICAL
