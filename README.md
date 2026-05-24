# The Temperature Box: An Introduction to Control Systems Project

[![Platform: Arduino](https://img.shields.io/badge/Platform-Arduino-00979D?style=flat-square&logo=arduino&logoColor=white)](https://www.arduino.cc/)
[![Course: Control Systems](https://img.shields.io/badge/Course-Control%20Systems%20%28EEE305%29-blue?style=flat-square)](#)
[![Institution: BRAC University](https://img.shields.io/badge/Institution-BRAC%20University-002F6C?style=flat-square)](#)

A comprehensive implementation and evaluation of a **Closed-Loop Temperature Control System** exploring both **Analog (Hardware-driven)** and **Digital (Microcontroller-driven)** frameworks. This project applies foundational feedback control principles to regulate temperature within an established boundary using hysteresis logic to eliminate oscillation and relay chatter.

---

## 📌 Project Objectives
* [cite_start]**Implement Closed-Loop Systems:** Apply practical feedback control loops utilizing both digital and analog processing methods[cite: 6, 8].
* [cite_start]**Temperature Sensing:** Interface K-type thermocouples via discrete op-amp signal conditioning circuits vs. specialized SPI modules (MAX6675)[cite: 7, 21].
* [cite_start]**Hysteresis Window Optimization:** Develop control systems capable of maintaining steady-state temperature windows without component wear (relay chattering)[cite: 36, 76, 77].
* [cite_start]**Engineering Trade-off Analysis:** Perform comparative analysis gauging accuracy, noise immunity, component counts, flexibility, and cost-efficiency[cite: 9, 10, 96].

---

## 🏗️ System Architecture & Mechanics

### 🔌 1. Digital Controller Design
[cite_start]The digital implementation utilizes an **Arduino Uno** running software-defined logic[cite: 13]. [cite_start]It interfaces with a **MAX6675** cold-junction compensated ADC to capture real-time readings from a K-type thermocouple over a 3-wire Software SPI interface[cite: 13, 20, 21, 37].

#### 🛠️ Pin Mapping & Configuration
| MAX6675 / Relay Pin | Arduino Pin | Function Description |
| :--- | :---: | :--- |
| **SO** (Serial Out) | Pin 4 | [cite_start]Sends digital data bytes to the controller [cite: 21] |
| **CS** (Chip Select) | Pin 5 | [cite_start]Enables/disables SPI bus communication [cite: 21] |
| **SCK** (Serial Clock) | Pin 6 | [cite_start]Synchronizes serial data timing [cite: 21] |
| **IN** (Relay Trigger) | Pin 8 | [cite_start]Actuates the high-power load (Lamp) [cite: 24, 26] |

#### 🔄 Hysteresis Control Parameters
* [cite_start]**Upper Threshold:** $\ge 33^\circ\text{C} \rightarrow$ Light **OFF** [cite: 31]
* [cite_start]**Lower Threshold:** $\le 25^\circ\text{C} \rightarrow$ Light **ON** [cite: 32]
* [cite_start]**Buffer Zone:** $25^\circ\text{C} - 33^\circ\text{C} \rightarrow$ **Maintains the previous latch state** to mitigate rapid switching[cite: 33, 36].

---

### 🎛️ 2. Analog Controller Design
[cite_start]A completely solid-state, hardware-based closed-loop architecture eliminating software runtimes[cite: 38]. [cite_start]It leverages multi-stage cascading operational amplifier configurations to parse signals[cite: 39, 98]:

1. [cite_start]**Difference Amplifier Stage (U1):** Rejects common-mode noise and boosts the raw microvolt ($\mu\text{V}/^\circ\text{C}$) thermocouple signal into readable potential voltage[cite: 41, 55, 57].
2. [cite_start]**Low-Pass Filter & Inverting Stage (U2):** A first-order RC LPF attenuates electromagnetic high-frequency noise and stabilizes sensor readings[cite: 62, 63, 64].
3. [cite_start]**Dual Window Comparators (U3 + U4):** Adjustable potentiometers map the upper voltage limit ($V_H$) and lower voltage limit ($V_L$) thresholds[cite: 44, 45, 71, 74].
4. [cite_start]**SR Latch Circuit (U5 + U6):** Two cross-coupled NOR gates form a hardware state memory bank to capture threshold transitions and inject hysteresis natively[cite: 46, 78, 79].
5. [cite_start]**Relay Driver Circuit (Q1):** An NPN transistor amplifies the low-current logic latch output to engage the relay coil, accompanied by a flyback diode to suppress inductive back-EMF spikes[cite: 48, 88, 89, 92, 93].

---

## 💻 Source Code (Digital Controller)

[cite_start]The following software routine processes the SPI data structure from the thermocouple module and drives the power loop[cite: 13, 21].

```cpp
#include <max6675.h>

// MAX6675 thermocouple pins
int thermoSO = 4;
int thermoCS = 5;
int thermoSCK = 6;

// Relay control pin
int relayPin = 8;

// Create MAX6675 object
MAX6675 thermocouple(thermoSCK, thermoCS, thermoSO);

// Light state variable (stores previous ON/OFF state)
bool lightOn = false;

void setup() {
  Serial.begin(9600);
  pinMode(relayPin, OUTPUT);
  digitalWrite(relayPin, HIGH); // Relay OFF (light OFF) at startup
}

void loop() {
  float temp = thermocouple.readCelsius();
  
  Serial.print("Temperature: ");
  Serial.print(temp);
  Serial.print(" C | ");

  // ===== HYSTERESIS CONTROL LOGIC =====
  if (temp >= 33.0) {
    lightOn = false; // Over-temperature limit crossed
  } 
  else if (temp <= 25.0) {
    lightOn = true;  // Under-temperature limit crossed
  }
  // Keeps previous lightOn state if within the 25°C - 33°C window

  // Apply relay output based on computed state (Active LOW relay module)
  if (lightOn) {
    digitalWrite(relayPin, LOW);  // Relay ON -> Heater element ON
    Serial.println("LIGHT ON");
  } else {
    digitalWrite(relayPin, HIGH); // Relay OFF -> Heater element OFF
    Serial.println("LIGHT OFF");
  }
  
  delay(1000); // Poll environmental states every 1 second
}
