# ⚡ Smart IoT-Based Energy Management System

An intelligent **IoT-based energy management and safety system** designed to reduce energy wastage, monitor electrical parameters in real time, and automatically disconnect appliances during unsafe operating conditions.

The system combines **ESP32 edge computing, real-time energy monitoring, occupancy-based automation, Firebase cloud synchronization, React/Flutter interfaces, software fault injection, and context-aware AI diagnostics** into a single end-to-end platform.

---

## 🚀 Key Features

* Real-time monitoring of **voltage, current, power, and energy**
* Temperature and humidity monitoring
* PIR-based **occupancy detection**
* Automatic appliance ON/OFF control
* Manual control through web/mobile interface
* Local **overvoltage and temperature safety cutoff**
* Fail-safe relay configuration
* Real-time Firebase synchronization
* Software-based voltage fault injection for safety testing
* React Web and Flutter Mobile interfaces
* Context-aware Gemini AI assistant
* Edge-based safety decisions independent of cloud availability

---

## 🏗️ System Architecture

```text
             ┌─────────────────────┐
             │     AC Appliance    │
             └──────────┬──────────┘
                        │
                     Relay
                        │
                        ▼
┌──────────┐      ┌──────────┐      ┌──────────┐
│  PZEM    │─────▶│          │◀─────│  DHT22   │
│ Energy   │ UART │   ESP32  │      │ Temp/Hum. │
│  Meter   │      │          │      └──────────┘
└──────────┘      │  Edge    │
                  │ Controller│◀────┐
                  └────┬─────┘     │
                       │           │
                    Wi-Fi        PIR
                       │        Motion
                       ▼
             ┌──────────────────┐
             │ Firebase RTDB     │
             └────────┬─────────┘
                      │
             ┌────────┴─────────┐
             ▼                  ▼
       React Web App       Flutter App
             │
             ▼
       Gemini AI Assistant
```

---

# 🔧 Hardware

| Component                       | Purpose                                           |
| ------------------------------- | ------------------------------------------------- |
| **ESP32**                       | Main edge controller                              |
| **PZEM-004T V3.0**              | AC voltage, current, power and energy measurement |
| **DHT22**                       | Temperature and humidity sensing                  |
| **PIR Sensor**                  | Occupancy/motion detection                        |
| **5V Active-Low Relay**         | Appliance control and safety cutoff               |
| **HLK-PM01 / HLK AC-DC Module** | Isolated AC-to-DC power conversion                |
| AC Load                         | Controlled electrical appliance                   |

---

# 📌 ESP32 Pin Configuration

| Component | ESP32 Pin | Function       |
| --------- | --------: | -------------- |
| PZEM RX   |   GPIO 16 | UART2 RX       |
| PZEM TX   |   GPIO 17 | UART2 TX       |
| Relay     |   GPIO 26 | Digital Output |
| DHT22     |   GPIO 14 | Digital Input  |
| PIR       |   GPIO 13 | Digital Input  |

> **Note:** Pin assignments should always be verified against the actual hardware wiring and ESP32 development board being used.

---

# ⚙️ Embedded Software

The ESP32 firmware is written in **C++ using the Arduino framework**.

The controller continuously performs four major tasks:

1. Reads sensor data.
2. Evaluates safety conditions.
3. Controls the relay.
4. Synchronizes data with Firebase.

### Non-Blocking Control

The system uses `millis()`-based timing instead of continuously using `delay()` for runtime control.

For example, the PIR automation keeps the appliance ON while motion is detected and switches it OFF after **7 seconds without detected motion**.

This allows sensor monitoring, relay control, and cloud communication to operate without unnecessarily blocking the main loop.

---

# 🛡️ Safety Logic

Safety decisions are performed locally on the ESP32.

The control priority is:

```text
1. Safety Cutoff
       ↓
2. Manual Override
       ↓
3. Automatic PIR Control
```

The system disconnects the load when:

```text
Temperature > 30°C
OR
Voltage >= 250V
```

When a safety condition occurs:

```text
Abnormal Condition
        ↓
ESP32 detects fault
        ↓
Relay → OFF
        ↓
Safety status → Firebase
        ↓
Frontend displays warning
```

This means the system does **not depend on cloud communication for the primary safety decision**.

---

# 🔌 PZEM-004T Communication

The PZEM-004T communicates with the ESP32 through **UART2**.

```text
PZEM TX ───────▶ ESP32 GPIO16 (RX2)

PZEM RX ◀─────── ESP32 GPIO17 (TX2)
```

The PZEM provides:

* Voltage
* Current
* Power
* Energy

The ESP32 reads these values and synchronizes them with Firebase.

---

# ☁️ Firebase Architecture

The system uses **Firebase Realtime Database (RTDB)** for cloud synchronization.

The database is logically divided into:

```text
IoT_Hub/
│
├── sensor_data/
│   ├── voltage
│   ├── current
│   ├── power
│   ├── energy
│   ├── temperature
│   ├── humidity
│   └── motion
│
├── status/
│   ├── relayState
│   └── safetyCutoff
│
└── commands/
    ├── manual_override
    ├── manual_relay_state
    └── simulate_voltage_spike
```

The ESP32 uploads sensor readings and system status while the frontend can write control commands.

---

# 🧪 Software Fault Injection

One of the major testing features is **software-based fault injection**.

Instead of physically applying an unsafe voltage to the system, the frontend can activate:

```text
simulate_voltage_spike = true
```

The firmware then simulates:

```cpp
voltage = 300.0;
```

The safety system processes this exactly like an abnormal voltage condition.

```text
Frontend
   ↓
simulate_voltage_spike = true
   ↓
ESP32
   ↓
Simulated voltage = 300V
   ↓
Safety threshold exceeded
   ↓
Relay OFF
   ↓
safetyCutoff = true
   ↓
Firebase
   ↓
Frontend warning
```

This allows the safety logic to be tested without repeatedly exposing the physical system to dangerous electrical conditions.

---

# 🖥️ Frontend

The project supports both:

### React Web Application

Provides:

* Live telemetry dashboard
* Relay control
* Auto/Manual mode
* Safety status
* Fault injection controls
* AI assistant

### Flutter Mobile Application

Provides similar functionality through a mobile interface for remote monitoring and control.

---

# 🤖 Context-Aware AI Assistant

The system integrates **Google Gemini** as an intelligent diagnostic assistant.

Instead of sending only the user's question to the AI, the application also provides relevant live system information.

For example:

```text
User:
"Why did the power turn off?"

Live Context:
Voltage = 300V
Relay = OFF
Safety Cutoff = TRUE
```

The AI can then provide a grounded response such as:

```text
"The appliance was disconnected because the system
detected an overvoltage condition of 300V."
```

This makes the AI aware of the current physical state of the IoT system.

---

# 🔄 Complete Data Flow

```text
Physical Environment
        │
        ▼
 Sensors
        │
        ▼
      ESP32
        │
 ┌──────┴───────┐
 │              │
 ▼              ▼
Safety       Automation
Logic          Logic
 │              │
 └──────┬───────┘
        ▼
      Relay
        │
        ▼
    AC Appliance

        ESP32
          │
         Wi-Fi
          │
          ▼
     Firebase RTDB
          │
    ┌─────┴─────┐
    ▼           ▼
 React       Flutter
    │
    ▼
 Gemini AI
```

---

# 🧠 Engineering Concepts Demonstrated

This project demonstrates practical knowledge of:

* Embedded C/C++
* ESP32 architecture
* GPIO
* UART communication
* Sensor interfacing
* Relay control
* Edge computing
* Non-blocking programming
* Real-time monitoring
* IoT architecture
* Firebase RTDB
* Wi-Fi communication
* Cloud-to-device commands
* Fault injection testing
* HIL/SIL testing concepts
* React
* Flutter
* REST/cloud concepts
* AI/LLM integration
* Context-aware diagnostics
* Fail-safe embedded design

---

# 🔮 Future Improvements

Possible extensions include:

1. **Energy Consumption Analytics**
   Add historical data analysis, daily/monthly consumption reports, and energy-saving recommendations.

2. **Predictive Fault Detection**
   Use machine learning to identify abnormal voltage, current, temperature, or power-consumption patterns before a failure occurs.

3. **Industrial/Automotive Expansion**
   Extend the architecture to multiple loads, industrial sensors, CAN-based communication, and more advanced safety and diagnostic mechanisms.

---

# 🎯 Project Objective

The primary objective is to develop an energy management platform that can **monitor electrical parameters, reduce unnecessary energy consumption, automatically respond to unsafe conditions, and provide users with real-time visibility and intelligent diagnostics**.

The project demonstrates how **edge computing and IoT can work together**, with the ESP32 responsible for time-critical local decisions while Firebase and frontend applications provide remote monitoring, control, and visualization.

---

## 👨‍💻 Role

**Lead Engineer — Sagar Anil Shigehalli**

Responsible for the system architecture, ESP32 firmware, sensor and relay integration, safety logic, Firebase communication, fault-injection testing, and integration of the monitoring/control interfaces.

---

## 🛠️ Technology Stack

**Hardware:**
ESP32 · PZEM-004T V3.0 · DHT22 · PIR · Relay · HLK AC-DC Module

**Embedded:**
C++ · Arduino IDE · UART · GPIO · Wi-Fi · Non-blocking timing

**Cloud:**
Firebase Realtime Database · Firebase Authentication

**Frontend:**
React · Flutter

**AI:**
Google Gemini API

**Testing:**
Software Fault Injection · HIL/SIL Concepts

---

## ⚠️ Safety Notice

This project involves **AC mains voltage**, which can cause serious injury or death. High-voltage wiring, isolation, relay connections, and power-supply integration should only be performed using appropriate electrical safety practices and by qualified personnel.
