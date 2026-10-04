# Dashboard – Smart Grain Storage Monitoring System

## Overview

This folder contains the dashboard implementation for the **Smart Grain Storage and Mushroom Monitoring System for Sri Lankan grain storage facilities**.

The dashboard receives environmental data from multiple ESP32-based wireless sensor nodes through **Wi-Fi and MQTT** and provides centralized real-time monitoring using **Node-RED**.

The main purpose of the dashboard is to help identify storage conditions that may increase the risk of fungal growth and grain deterioration by monitoring parameters such as:

- Temperature
- Relative humidity
- Temperature variation over time
- Battery status
- Wi-Fi signal strength
- Node connectivity
- Risk level
- Alarm condition

> The system does not directly identify or confirm the presence of fungi.  
> It identifies environmental conditions and trends that may indicate an increased risk of fungal growth and grain deterioration.

---

## System Architecture

```text
        SMART GRAIN STORAGE MONITORING SYSTEM

 ┌──────────────────────┐
 │     Sensor Node 01   │
 │ Temperature          │
 │ Humidity             │
 │ NTC Temperature      │
 │ Battery Monitoring   │
 └──────────┬───────────┘
            │
            │
 ┌──────────▼───────────┐
 │     Sensor Node 02   │
 │ Temperature          │
 │ Humidity             │
 │ NTC Temperature      │
 │ Battery Monitoring   │
 └──────────┬───────────┘
            │
            │
 ┌──────────▼───────────┐
 │     Sensor Node N    │
 └──────────┬───────────┘
            │
            │ Wi-Fi
            ▼
 ┌──────────────────────┐
 │ Wi-Fi Router / AP    │
 └──────────┬───────────┘
            │
            ▼
 ┌──────────────────────┐
 │     MQTT Broker      │
 └──────────┬───────────┘
            │
            ▼
 ┌──────────────────────┐
 │       Node-RED       │
 │      Dashboard       │
 └──────────┬───────────┘
            │
   ┌────────┼───────────────┐
   │        │               │
   ▼        ▼               ▼
Live Data  Trends        Alerts
   │
   ▼
Storage Zone Monitoring
