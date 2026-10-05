# Temperature Node – Pinout / Connection List

---

## 1. ESP-12F / ESP8266 Main Pinout
__________________________________________________________________________________________________________________________
| ESP-12F Pin | ESP-12F Signal | PCB Connection    | Connected Component / Net            | External / Manual Connection |
|-------------|----------------|-------------------|--------------------------------------|------------------------------|
| 1           | RST            | Reset circuit     | R9.1, SW4.1/SW4.2, 10 kΩ pull-up     |                              |
| 2           | ADC            | External header   | R2.1, **H2.5**                       | NTC 10 kohm                  |
| 3           | EN             | Pull-up           | R5.1, 10 kΩ pull-up network          |                              |
| 4           | IO16           | Pull-up           | R6.1, 10 kΩ pull-up                  |                              |
| 5           | IO14           | NC                | Not connected in schematic           |                              |
| 6           | IO12           | NC                | Not connected in schematic           |                              |
| 7           | IO13           | NC                | Not connected in schematic           |                              |
| 8           | VCC            | 3.3 V rail        | U7 VOUT, C2/C3, CN4.4, H2.2          |                              |
| 9           | CS0            | NC                | Not connected in schematic           |                              |
| 10          | MISO           | NC                | Not connected in schematic           |                              |
| 11          | IO9            | Buzzer            | U8.1                                 |                              |
| 12          | IO10           | NC                | Not connected in schematic           |                              |
| 13          | MOSI           | NC                | Not connected in schematic           |                              |
| 14          | SCLK           | MOSFET gate drive | R3.2 → R3 100 Ω → Q1 gate            |                              |
| 15          | GND            | Ground            | GND / H2.1 / H2.4 / U5.2 / SW3.4     |                              |
| 16          | IO15           | Boot pull-down    | R8.2, R8.1 → GND                     |                              |
| 17          | IO2            | Pull-up           | R7.1, 10 kΩ pull-up                  |                              |
| 18          | IO0            | Programming / boot| R7.1, U5.1; jumper to GND through U5 |                              |
| 19          | IO4            | External header   | **H2.3**                             | DS18B20 Temparature sensor   |
| 20          | IO5            | Push button       | SW3.1; SW3.4 → GND                   |                              |
| 21          | RXD            | External header   | **CN4.3**                            | Header for programming & GPS |
| 22          | TXD            | External header   | **CN4.2**                            | Header for programming & GPS |

---

## 2. External Connector CN4 – 4 Pin Header

**Part:** HC-EH2.54-4A-TW  
**PCB footprint:** 1×4, 2.5 mm pitch
______________________________________________________________________________
| CN4 Pin | PCB Net / Connection              | Manual External Function     |
|---------|-----------------------------------|------------------------------|
| 1       | Q1 Drain / MOSFET switched output | GND of GPS (optional)        |
| 2       | ESP-12F TXD (Pin 22)              | Header for programming & GPS |
| 3       | ESP-12F RXD (Pin 21)              | Header for programming & GPS |
| 4       | 3.3 V / ESP-12F VCC               | VCC of GPS (optional)        |



---

## 3. External Header H2 – 5 Pin Header

**Part:** 1×5, 2.54 mm pitch
_____________________________________________________________
| H2 Pin | PCB Connection        | Manual External Function |
|--------|-----------------------|--------------------------|
| 1      | GND                   | GND of DS18B20           |
| 2      | 3.3 V                 | VCC of DS18B20           |
| 3      | ESP-12F IO4 (Pin 19)  | Input pin for DS18B20    |
| 4      | GND                   | GND of NTC               |
| 5      | ESP-12F ADC (Pin 2)   | Input pin for NTC data   |

---

## 4. Power Connector CN5 – 2 Pin

**Part:** JST-PH-2-SMT-RA
___________________________________
| CN5 Pin | PCB Connection        |
|---------|-----------------------|
| 1       | U7 VIN / input supply |
| 2       | GND                   |

### Power path

`CN5.1 → U7 VIN → AMS1117-3.3 → 3.3 V`

`CN5.2 → GND`

---

## 5. Programming Jumper U5

**Part:** 2-pin jumper
__________________________________________________________________________
| U5 Pin | Connection           | Function                               |
|--------|----------------------|----------------------------------------|
| 1      | ESP-12F IO0 / Pin 18 | Short circuited while upload programme |
| 2      | GND                  | Short circuited while upload programme |

### Programming operation

For ESP8266 UART programming:

**U5 ON / shorted:** IO0 → GND

**U5 OFF / open:** IO0 pulled HIGH for normal boot

---

## 6. Push Button for getting GPS locatiion SW3
___________________________________________________
| SW3 Pin | Connection                            |
|---------|---------------------------------------|
| 1       | ESP-12F IO5 / Pin 20                  |
| 2       | Not explicitly connected in schematic |
| 3       | Not explicitly connected in schematic |
| 4       | GND                                   |

---

## 7. Reset Button SW4
___________________________________________________
| SW4 Pin | Connection                            |
|---------|---------------------------------------|
| 1       | ESP-12F RST / Pin 1                   |
| 2       | ESP-12F RST / Pin 1                   |
| 3       | GND                                   |
| 4       | Not explicitly connected in schematic |



---

## 8. MOSFET Q1

**Part:** IRLZ44PBF
_______________________________________________________________
| Q1 Pin | MOSFET Pin Name | PCB Connection                   |
|--------|-----------------|----------------------------------|
| 1      | Gate (G)        | R3.1 → R3 100 Ω → ESP-12F Pin 14 |
| 2      | Drain (D)       | CN4.1                            |
| 3      | Source (S)      | GND through R4/R8 ground network |

---

## 9. Buzzer U8

**Part:** HX-2212
_________________________________
| U8 Pin | Connection           |
|--------|----------------------|
| 1      | ESP-12F IO9 / Pin 11 |
| 2      | GND                  |

---

## 10. Voltage Regulator U7

**Part:** AMS1117-3.3
__________________________________
| U7 Pin | Function | Connection |
|--------|----------|------------|
| 1      | GND      | GND        |
| 2      | VOUT     | 3.3 V rail |
| 3      | VIN      | CN5.1      |
| 4      | VOUT     | 3.3 V rail |

---

## 11. Important Pull-Up / Pull-Down Resistors
______________________________________________________
| Resistor | Value   | Connection                    |
|----------|---------|-------------------------------|
| R2       | 10 kΩ   | 3.3 V → ESP ADC               |
| R3       | 100 Ω   | ESP Pin 14 → MOSFET gate      |
| R4       | 10 kΩ   | MOSFET gate → GND             |
| R5       | 10 kΩ   | ESP EN pull-up                |
| R6       | 10 kΩ   | ESP IO16 pull-up              |
| R7       | 10 kΩ   | ESP IO2 / IO0 pull-up network |
| R8       | 10 kΩ   | ESP IO15 pull-down            |
| R9       | 10 kΩ   | ESP RST pull-up               |

---

## 12. Capacitors
______________________________________________
| Capacitor | Value | Connection             |
|-----------|-------|------------------------|
| C1        | 10 µF | Input supply filtering |
| C2        | 10 µF | 3.3 V output filtering |
| C3        | 100 nF| 3.3 V decoupling       |

---

# 14. ESP8266 GPIO Usage Summary
___________________________________________________________
| GPIO          | ESP-12F Pin | Current Use               |
|---------------|-------------|---------------------------|
| GPIO0         | 18          | Programming / boot jumper |
| GPIO1 / TXD   | 22          | UART TX → CN4.2           |
| GPIO2         | 17          | Pull-up / boot            |
| GPIO3 / RXD   | 21          | UART RX ← CN4.3           |
| GPIO4         | 19          | H2.3 external             |
| GPIO5         | 20          | SW3 button                |
| GPIO6         | 9           | Flash interface / CS0     |
| GPIO7         | 13          | Flash interface / MOSI    |
| GPIO8         | 12          | Flash interface / IO10    |
| GPIO9         | 11          | Buzzer                    |
| GPIO10        | 12          | Flash interface / IO10    |
| GPIO12        | 6           | NC                        |
| GPIO13        | 7           | NC                        |
| GPIO14        | 5           | NC                        |
| GPIO15        | 16          | Boot pull-down            |
| GPIO16        | 4           | Pull-up                   |
| ADC0          | 2           | H2.5 external ADC         |
| RST           | 1           | Reset switch              |
| EN            | 3           | 10 kΩ pull-up             |
| VCC           | 8           | 3.3 V                     |
| GND           | 15          | Ground                    |

---


