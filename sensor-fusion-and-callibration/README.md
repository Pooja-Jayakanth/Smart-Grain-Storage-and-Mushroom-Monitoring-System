# Sensor Fusion Pipeline — Semi-Finals Prototype


## 1. Sensor-Fusion Objective

The purpose of the sensor-fusion subsystem is to obtain a temperature estimate that is more reliable than depending on only one sensor.

The two temperature sensors use different measurement principles:

- **DS18B20** — digital temperature sensor using the 1-Wire interface.
- **NTC thermistor** — analog resistive temperature sensor read using the ESP32 ADC through a voltage-divider circuit.

Because the two sensors have different levels of noise, calibration error, and measurement uncertainty, they are **not treated as equally reliable**.

The processing method was developed in stages.

### Initial Development

```text
Raw Sensor Measurement
        ↓
EMA Filtering
        ↓
Filtered Sensor Measurement
```

### Final Implemented Processing

```text
Sensor Acquisition
        ↓
Calibration
        ↓
Range Validation
        ↓
Outlier Rejection
        ↓
Sensor Validity Check
        ↓
Weighted Least Squares (WLS) Fusion
        ↓
Fused Temperature + Fused Variance
        ↓
Scalar Kalman Filter
        ↓
Final Temperature Estimate
```

---

## 2. Sensors Used

### 2.1 DS18B20

The DS18B20 provides a direct digital temperature value to the ESP32.

**Main features**

- Digital temperature measurement
- 1-Wire communication
- Does not require the ESP32 ADC
- Lower measured noise than the NTC sensor
- Provides one independent temperature measurement for fusion
- Uses a **4.7 kΩ pull-up resistor** on the data line

The DS18B20 path is:

```text
DS18B20
   ↓
Digital Temperature Reading
   ↓
Calibration
   ↓
Validation
   ↓
Outlier Rejection
   ↓
WLS Fusion
```

### 2.2 MF5A-3 10 kΩ NTC Thermistor

The MF5A-3 is the analog temperature sensor used in the prototype.

The thermistor is connected as a voltage divider:

```text
3.3 V
  │
10 kΩ Fixed Resistor
  │
  ├── ESP32 ADC
  │
10 kΩ NTC
  │
 GND
```

Parameters used:

| Parameter | Value |
|---|---:|
| Nominal resistance, R₀ | 10 kΩ |
| Reference temperature, T₀ | 298.15 K |
| Beta constant, B | 3950 K |
| ESP32 ADC resolution | 12 bit |
| Maximum ADC count | 4095 |

The NTC path is:

```text
NTC Voltage Divider
        ↓
ADC Sampling
        ↓
20-Sample Averaging
        ↓
Resistance Calculation
        ↓
Beta Equation
        ↓
Raw NTC Temperature
        ↓
Linear Calibration
        ↓
Validation
        ↓
Outlier Rejection
        ↓
WLS Fusion
```

---

## 3. NTC ADC Averaging

The NTC ADC reading is averaged over **20 samples** before converting it to resistance.

The 20 ADC samples are:

**ADC₁, ADC₂, ADC₃, …, ADC₂₀**

The average ADC value is:

**ADC_avg = (ADC₁ + ADC₂ + ADC₃ + … + ADC₂₀) / 20**

This averaged value is used in the resistance calculation.

---

## 4. NTC Resistance Calculation

The NTC resistance is calculated from the voltage-divider output as:

**R_NTC = R_fixed × ADC_avg / (ADC_max − ADC_avg)**

where:

- **R_fixed = 10,000 Ω**
- **ADC_max = 4095**

for the 12-bit ESP32 ADC.

---

## 5. NTC Resistance-to-Temperature Conversion

The measured NTC resistance is converted to temperature using the Beta equation:

**1 / T_K = 1 / T₀ + (1 / B) × ln(R_NTC / R₀)**

where:

- **T₀ = 298.15 K**
- **R₀ = 10,000 Ω**
- **B = 3950 K**

The raw NTC temperature in degrees Celsius is:

**T_NTC,raw = T_K − 273.15**

---

## 6. Sensor Calibration

Both temperature sensors were experimentally compared against a reference thermometer at:

- 28.0 °C
- 29.0 °C
- 29.5 °C
- 31.0 °C

The calibration stage was used to distinguish **systematic measurement error** from **random measurement variation**.

### 6.1 DS18B20 Calibration

The DS18B20 did not show a consistent fixed positive or negative bias across the tested temperatures.

Therefore, no additional linear correction was applied.

**T_DS,cal = T_DS,raw**

The calibration coefficients are therefore:

- **A_DS = 1**
- **B_DS = 0**

### 6.2 NTC Calibration

The NTC showed a systematic positive offset and therefore required linear calibration.

The general calibration equation is:

**T_cal = A × T_raw + B_c**

The experimentally obtained coefficients for the NTC are:

- **A_NTC = 1.008076**
- **B_NTC = −2.507173**

Therefore:

**T_NTC,cal = 1.008076 × T_NTC,raw − 2.507173**

This calibrated NTC temperature is used for fusion.

---

## 7. Initial EMA Filtering Stage

During the initial development stage, an **Exponential Moving Average (EMA)** filter was tested to reduce short-term fluctuations in the individual sensor measurements.

The EMA equation is:

**T_EMA(k) = α × T(k) + (1 − α) × T_EMA(k−1)**

where:

- **T(k)** = current sensor measurement
- **T_EMA(k−1)** = previous filtered value
- **T_EMA(k)** = new filtered value
- **α** = smoothing coefficient

The value used during testing was:

**α = 0.20**

Therefore:

**T_EMA(k) = 0.20 × T(k) + 0.80 × T_EMA(k−1)**

The initial development flow was:

```text
DS18B20 Raw
    ↓
EMA
    ↓
Filtered DS18B20

NTC Raw
    ↓
EMA
    ↓
Filtered NTC
```

EMA was useful because it:

- required very little memory,
- required low computation,
- reduced sample-to-sample fluctuations,
- and was simple to implement on the ESP32.

However, EMA only performs **temporal smoothing**. It does not determine how much each sensor should be trusted relative to the other.

For that reason, EMA was not used as the final fusion method.

---

## 8. Measurement Validation

Before sensor fusion, each calibrated temperature is checked for validity.

The prototype applies two main checks.

### 8.1 Range Validation

The selected valid operating range is:

**−20 °C < T < 100 °C**

Measurements at or outside the limits are rejected.

### 8.2 Sudden-Change Rejection

A new measurement is rejected if the change from the previous accepted measurement is greater than:

**8 °C**

That is:

**|T(k) − T(k−1)| > 8 °C**

is treated as an invalid sudden change.

This check helps reject:

- disconnected-sensor readings,
- transient ADC disturbances,
- conversion errors,
- unrealistic spikes.

---

## 9. Sensor Noise Characterization

The measured standard deviations from the 100-sample characterization test were:

- **σ_DS = 0.0251 °C**
- **σ_NTC = 0.0540 °C**

The DS18B20 therefore showed lower random variation than the NTC.

These measured values are used directly in the WLS fusion algorithm.

---

## 10. Weighted Least Squares Sensor Fusion

The final fusion method is **inverse-variance Weighted Least Squares (WLS)**.

The main idea is:

> A sensor with lower experimentally measured variance is given a larger weight.

The variance of each sensor is:

**σ²_DS = (0.0251)² = 0.000630**

**σ²_NTC = (0.0540)² = 0.002916**

### 10.1 Information Values

For each sensor, the information value is the inverse of its variance:

**I_DS = 1 / σ²_DS**

**I_NTC = 1 / σ²_NTC**

A lower sensor variance therefore produces a larger information value.

### 10.2 WLS Weights

The information values are normalized to obtain the sensor weights:

**w_DS = I_DS / (I_DS + I_NTC)**

**w_NTC = I_NTC / (I_DS + I_NTC)**

The experimentally derived weights are approximately:

- **w_DS ≈ 0.822**
- **w_NTC ≈ 0.178**

and:

**w_DS + w_NTC = 1**

This means that the DS18B20 contributes more strongly because it showed lower measured noise.

### 10.3 Fused Temperature

The WLS fused temperature is:

**T_WLS = w_DS × T_DS + w_NTC × T_NTC**

Using the experimentally obtained weights:

**T_WLS = 0.822 × T_DS + 0.178 × T_NTC**

Therefore:

- DS18B20 contribution ≈ 82.2%
- NTC contribution ≈ 17.8%

The NTC is still retained as an independent second temperature measurement, providing redundancy.

---

## 11. Fused Measurement Variance

WLS also produces an uncertainty value for the fused measurement.

The fused variance is:

**R_WLS = 1 / [(1 / σ²_DS) + (1 / σ²_NTC)]**

This value represents the uncertainty of the WLS fused temperature.

The WLS stage therefore produces two important outputs:

- **T_WLS** — fused temperature
- **R_WLS** — fused measurement variance

The fused variance is passed directly to the Kalman filter as the measurement-noise term.

---

## 12. Fault-Tolerant Fusion

The fusion stage checks the validity of both temperature sensors.

### Both Sensors Valid

Normal WLS fusion is used:

**T_WLS = 0.822 × T_DS + 0.178 × T_NTC**

### Only DS18B20 Valid

- **w_DS = 1**
- **w_NTC = 0**

Therefore:

**T_WLS = T_DS**

### Only NTC Valid

- **w_DS = 0**
- **w_NTC = 1**

Therefore:

**T_WLS = T_NTC**

### Both Sensors Invalid

The fusion result is declared invalid.

```text
DS18B20 Invalid
       +
NTC Invalid
       ↓
Fusion Invalid
       ↓
No false temperature output
```

This allows the system to continue operating when only one temperature sensor fails.

---

## 13. Kalman Filtering After WLS

WLS combines the two simultaneous temperature measurements at a single sampling instant.

The Kalman filter then uses the sequence of WLS results over time.

```text
WLS
 ↓
Combines the two sensors at one sampling instant

Kalman Filter
 ↓
Tracks and smooths the fused temperature over time
```

The Kalman filter provides temporal smoothing while considering the uncertainty generated by the WLS stage.

---

## 14. Kalman Filter State

The state being estimated is the true temperature:

**x(k) = T(k)**

---

## 15. Prediction Step

The predicted temperature is assumed to remain close to the previous estimate:

**x̂(k|k−1) = x̂(k−1|k−1)**

The predicted covariance is:

**P⁻(k) = P(k−1) + Q**

---

## 16. Process Noise

The process-noise standard deviation used in the prototype is:

**σ_Q = 0.10**

Therefore:

**Q = (0.10)² = 0.01**

This term represents expected real temperature variation between consecutive samples.

---

## 17. Measurement Noise

The measurement-noise term used by the Kalman filter is obtained directly from WLS:

**R(k) = R_WLS**

This directly connects the WLS stage to the Kalman filter.

---

## 18. Kalman Gain

The Kalman gain is calculated as:

**K(k) = P⁻(k) / [P⁻(k) + R(k)]**

Interpretation:

- Smaller **R(k)** → greater confidence in the new WLS measurement
- Larger **R(k)** → greater reliance on the predicted temperature

---

## 19. Kalman Measurement Update

The WLS fused temperature becomes the Kalman measurement input:

**z(k) = T_WLS**

The updated temperature estimate is:

**x̂(k) = x̂(k|k−1) + K(k) × [z(k) − x̂(k|k−1)]**

The covariance is then updated as:

**P(k) = [1 − K(k)] × P⁻(k)**

The resulting value **x̂(k)** is the final temperature estimate.

---

## 20. Complete Sensor-Fusion Flow

```text
                         DS18B20
                            │
                            ▼
                 Digital Temperature
                            │
                            ▼
                       Calibration
                  T_DS,cal = T_DS,raw
                            │
                            ▼
                    Range Validation
                            │
                            ▼
                    Outlier Rejection
                            │
                            ▼
                     Validity Check
                            │
                            │
                            ├───────────────┐
                            │               │
                            │               ▼
                            │          WLS SENSOR
                            │            FUSION
                            │               ▲
                            │               │
                            │               │
                         NTC Thermistor     │
                            │               │
                            ▼               │
                    ESP32 ADC Sampling      │
                            │               │
                            ▼               │
                    20-Sample Average       │
                            │               │
                            ▼               │
                  Resistance Calculation    │
                            │               │
                            ▼               │
                      Beta Equation         │
                            │               │
                            ▼               │
                     T_NTC,raw              │
                            │               │
                            ▼               │
                    Linear Calibration      │
                            │               │
                            ▼               │
                     T_NTC,cal              │
                            │               │
                            ▼               │
                    Range Validation        │
                            │               │
                            ▼               │
                    Outlier Rejection       │
                            │               │
                            ▼               │
                     Validity Check         │
                            │               │
                            └───────────────┘
                                    │
                                    ▼
                     Weighted Least Squares
                                    │
                         ┌──────────┴──────────┐
                         │                     │
                         ▼                     ▼
                    T_WLS                 R_WLS
                         │                     │
                         └──────────┬──────────┘
                                    ▼
                         Scalar Kalman Filter
                                    │
                                    ▼
                       Final Temperature Estimate
```

---

## 21. Development Flow from EMA to Final Fusion

```text
INITIAL DEVELOPMENT
───────────────────

Raw DS18B20 ──→ EMA ──→ Filtered DS18B20

Raw NTC ──────→ EMA ──→ Filtered NTC

Purpose:
Evaluate basic short-term noise reduction.


        ↓


SENSOR CHARACTERIZATION
───────────────────────

Reference Temperature Testing
        ↓
Calibration
        ↓
Noise Measurement
        ↓
Standard Deviation
        ↓
Sensor Variance


        ↓


FINAL IMPLEMENTATION
────────────────────

DS18B20 + NTC
        ↓
Sensor Acquisition
        ↓
NTC 20-Sample ADC Averaging
        ↓
NTC Temperature Conversion
        ↓
Calibration of Both Sensors
        ↓
Range Validation
        ↓
Sudden-Change / Outlier Rejection
        ↓
Sensor Validity Decision
        ↓
Inverse-Variance WLS Fusion
        ↓
T_WLS + R_WLS
        ↓
Scalar Kalman Filter
        ↓
Final Temperature Estimate
```

---

## 22. Why the Final Method Was Selected

The final architecture is:

```text
Calibration
    ↓
Validity Checking
    ↓
WLS Sensor Fusion
    ↓
Kalman Filtering
```

This approach was selected because:

- the two sensors have different measured uncertainties,
- WLS uses measured sensor variance rather than arbitrary weighting,
- the more reliable sensor automatically receives a larger weight,
- the second sensor still provides redundancy,
- one sensor can temporarily replace the other during a fault,
- WLS produces a fused measurement variance,
- the fused variance can be passed directly to the Kalman filter,
- the Kalman filter smooths the fused temperature over time,
- the complete method is computationally suitable for the ESP32,
- and the method is transparent and easy to justify experimentally.

---

## 23. Why EMA Was Replaced

EMA was useful for initial filtering.

Its equation is:

**T_EMA(k) = α × T(k) + (1 − α) × T_EMA(k−1)**

However, EMA uses a fixed coefficient **α**.

It does not use the experimentally measured variance of each sensor.

WLS instead uses:

**wᵢ ∝ 1 / σᵢ²**

so the sensor weights come directly from measured sensor performance.

Therefore:

```text
EMA
 ↓
Initial temporal smoothing investigation

WLS
 ↓
Final multi-sensor fusion

Kalman Filter
 ↓
Final temporal estimation and smoothing
```

---

## 24. Why Simple Averaging Was Not Used

A simple average would be:

**T_avg = (T_DS + T_NTC) / 2**

which assumes:

- **w_DS = 0.5**
- **w_NTC = 0.5**

This would incorrectly treat both sensors as equally reliable.

The measured standard deviations show that the sensors are not equally reliable.

Therefore, inverse-variance WLS was selected instead.

---

## 25. Why Median Fusion Was Not Used

Median fusion is useful when at least three redundant measurements are available.

This prototype uses only two independent temperature sensors.

Therefore, median-based voting cannot provide meaningful majority-based fault rejection.

---

## 26. Why Complementary Filtering Was Not Used

Complementary filtering is typically useful when two sensors provide complementary information over different frequency ranges.

In this system:

- DS18B20 measures temperature.
- NTC thermistor measures temperature.

Both sensors measure the same physical quantity over the same timescale.

Therefore, a complementary filter does not provide a clear advantage.

---

## 27. Why an Extended Kalman Filter Was Not Required

The NTC measurement starts with a nonlinear resistance-to-temperature relationship.

However, that nonlinearity is handled before sensor fusion:

```text
NTC Resistance
      ↓
Beta Equation
      ↓
Temperature
      ↓
Linear Calibration
```

By the time fusion begins:

```text
DS18B20 → Temperature in °C
NTC     → Temperature in °C
```

Both inputs are already temperature values in the same domain.

Therefore, a nonlinear estimator such as an Extended Kalman Filter is not required.

---

## 28. Key Prototype Parameters

| Parameter | Value |
|---|---:|
| DS18B20 sensor | Digital temperature sensor |
| NTC sensor | MF5A-3 10 kΩ |
| NTC fixed resistor | 10 kΩ |
| NTC Beta constant | 3950 K |
| ESP32 ADC resolution | 12 bit |
| ADC maximum | 4095 |
| NTC ADC averaging | 20 samples |
| Calibration points | 28.0, 29.0, 29.5, 31.0 °C |
| DS18B20 calibration | T_cal = T_raw |
| NTC calibration slope | 1.008076 |
| NTC calibration offset | −2.507173 |
| DS18B20 standard deviation | 0.0251 °C |
| NTC standard deviation | 0.0540 °C |
| DS18B20 WLS weight | ≈ 0.822 |
| NTC WLS weight | ≈ 0.178 |
| EMA test coefficient | 0.20 |
| Sudden-change rejection | > 8 °C |
| Validation range | −20 °C < T < 100 °C |
| Kalman process-noise standard deviation | 0.10 |
| Kalman process noise, Q | 0.01 |
| Kalman measurement noise | R(k) = R_WLS |

---

## 29. Final Sensor-Fusion Summary

For the **semi-finals prototype**, the complete temperature-estimation path is:

```text
DS18B20 + MF5A-3 NTC
        ↓
Sensor Acquisition
        ↓
NTC 20-Sample ADC Averaging
        ↓
Temperature Conversion
        ↓
Sensor Calibration
        ↓
Range Validation
        ↓
Outlier Rejection
        ↓
Sensor Validity / Fault Handling
        ↓
Inverse-Variance WLS Sensor Fusion
        ↓
Fused Temperature T_WLS
+
Fused Variance R_WLS
        ↓
Scalar Kalman Filter
        ↓
Final Temperature Estimate
```

The **EMA filter is retained as part of the development history**, while the **final implemented sensor-fusion architecture is WLS followed by scalar Kalman filtering**.
