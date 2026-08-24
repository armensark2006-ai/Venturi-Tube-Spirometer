# ESP32 Venturi Tube Volumetric Flow Meter

An embedded real-time air volumetric flow measurement system utilizing an ESP32 microcontroller, an XGZP6847A differential pressure sensor, and Bernoulli's principle. Fluid flow rate is visually represented using Pulse Width Modulation (PWM) LED feedback.

---

## Hardware Architecture & Specifications

### 1. Component List
* Microcontroller: ESP32 Development Board
* Pressure Sensor: XGZP6847A Differential Pressure Sensor (5 kPa, 5V, Bidirectional -5 kPa to +5 kPa)
* Voltage Divider: 3x 10k Ohm resistors (10k Ohm top leg, 20k Ohm bottom leg)
* Signal Filter: 100 nF (0.1 uF) ceramic capacitor
* Visual Output: Standard LED with 220 - 330 Ohm current-limiting resistor
* Pneumatic Coupling: Soft silicone tubing (3 mm ID) / tape-shimmed sleeve

### 2. Circuit Schematic & Pin Mapping

| ESP32 Pin | Connected Component | Function |
| :--- | :--- | :--- |
| **5V / VIN** | Sensor Pin 2 (VDD) & Pin 4 (VDD) | 5V Power Supply |
| **GND** | Sensor Pin 3 & 6, LED Cathode, Divider Bottom | System Ground Rail |
| **GPIO 34** | Voltage Divider Junction (10k / 20k Ohm) | ADC Input (0V-3.3V Range) |
| **GPIO 18** | LED Anode via 220 Ohm Resistor | PWM Brightness Output |

```text
               +5V Rail
                  |
         [ XGZP6847A Sensor ]
                  | (OUT: 0.5V - 4.5V)
                  +-------------------+
                                      |
                                  [ 10k Ohm ]
                                      |
  ESP32 GPIO 34 <--- +----------------+
                     |                |
                 [ 20k Ohm ]     [ 100nF Cap ]
                     |                |
                     +----------------+
                  |
               GND Rail

### Phase 3: IoT, Wireless Telemetry & Web Dashboard
Transitioning the device from local Serial output to a connected cloud node allows real-time remote monitoring, data logging, and automated alerting.

* **2.4 GHz Network Infrastructure:** Configure the ESP32 WiFi stack specifically for 2.4 GHz IEEE 802.11b/g/n routers or dedicated local access points.
* **MQTT Telemetry Pipeline:** Publish real-time JSON payloads to an MQTT broker (e.g., Mosquitto, HiveMQ) at 100ms intervals:
  ```json
  {
    "device_id": "esp32_venturi_01",
    "timestamp_ms": 1771958400000,
    "sensor_voltage_v": 2.85,
    "pressure_kpa": 1.12,
    "flow_rate_lmin": 378.4,
    "total_volume_l": 45.2
  }


Future Directions & Engineering Roadmap
Phase 1: Custom Printed Circuit Board (PCB) Design
Transitioning from breadboard prototyping to a dedicated PCB (via KiCad) will resolve signal instability, loose contacts, and physical clutter.

Integrated Voltage Dividers & Filtering: Replace discrete through-hole resistors with high-precision SMD components (0.1% tolerance 0805 resistors) and an onboard RC low-pass filter to minimize ADC thermal noise.

Power Management: Include a low-dropout linear regulator (LDO) such as an AMS1117-3.3/5.0 to isolate sensor supply voltage from noisy USB power rails.

Dedicated Connectors: Implement JST-XH or screw terminal connectors for pneumatic sensor inputs, LED output indicators, and external power supplies.

ESD & Reverse Polarity Protection: Add Schottky diodes and TVS diodes across inputs to prevent hardware damage during bench handling.

Plaintext
               PCB DEVELOPMENT ROADMAP
[ Breadboard Prototype ]  --->  [ KiCad Schematic & Layout ]
                                       |
                                       v
[ Production Assembly ]   <---  [ Gerber Generation & Fabrication ]
Phase 2: Mechanical Enclosure & Integrated Pneumatics
Custom 3D-Printed Housing: Design a snap-fit enclosure in CAD (Fusion 360) that securely holds the ESP32, PCB, and XGZP6847A sensor module.

Barbed Hose Fittings: Replace temporary tape/straw adapters with brass or 3D-printed 3 mm barbed fittings directly integrated into the Venturi tube throat for leak-free sealing.

Strain Relief: Incorporate cable glands and pneumatic tube anchor channels to prevent mechanical stresses on solder joints.

Phase 3: IoT, Wireless Telemetry & Web Dashboard
Transitioning the device from local Serial output to a connected cloud node allows real-time remote monitoring, data logging, and automated alerting.

2.4 GHz Network Infrastructure: Configure the ESP32 WiFi stack specifically for 2.4 GHz IEEE 802.11b/g/n routers or dedicated local access points.

MQTT Telemetry Pipeline: Publish real-time JSON payloads to an MQTT broker (e.g., Mosquitto, HiveMQ) at 100ms intervals:

JSON
{
  "device_id": "esp32_venturi_01",
  "timestamp_ms": 1771958400000,
  "sensor_voltage_v": 2.85,
  "pressure_kpa": 1.12,
  "flow_rate_lmin": 378.4,
  "total_volume_l": 45.2
}
Interactive Web Dashboard: Visualize live metrics using Grafana, Node-RED, or a WebSocket web front-end featuring:

Instantaneous Flow Rate (L/min) gauge

Differential Pressure (kPa) timeline plot

Total Accumulated Volume (Liters) calculated via numerical integration over time

Over-The-Air (OTA) Updates: Integrate ArduinoOTA to deploy firmware patches and recalibrate flow parameters wirelessly without tethering over USB.

Phase 4: Environmental Compensation & Sensor Fusion
Air density (rho) fluctuates with ambient temperature, pressure, and relative humidity. Upgrading from a constant air density assumption to live dynamic compensation increases accuracy under changing climate conditions.

I2C Environmental Sensor: Connect a BME280 sensor to read ambient temperature (T), relative humidity (RH), and absolute atmospheric pressure (P_abs).

Dynamic Air Density Calculation: Compute real-time air density inside the firmware loop using the ideal gas equation corrected for vapor pressure.

Mass Flow Rate: Compute true mass flow rate in grams per second (g/s) alongside volumetric flow rate.

Phase 5: Signal Processing & Noise Reduction
Pneumatic turbulence and ESP32 ADC jitter can create noisy raw readings. Software filtering stabilizes output values without introducing noticeable measurement latency.

Exponential Moving Average (EMA) / Discrete Kalman Filter: Implement a digital low-pass filter to smooth raw ADC voltage inputs.

Dynamic Zero-Drift Tracking: Automatically update the baseline V_zero offset when the system detects sustained zero-flow conditions.

Hysteretic Dead-Band: Apply threshold limits near zero flow to eliminate flickering readings caused by acoustic micro-vibrations.

Phase 6: Calibration & Edge Analytics
Multi-Point Discharge Coefficient Mapping: Replace the ideal Bernoulli equation with a dynamic discharge coefficient (Cd) mapped across Reynolds numbers (Re).

On-Device Anomaly Detection: Flag operational hardware faults in real time:

Tube Leak / Disconnection: Zero pressure drop recorded during high fan/blower drive states.

Pneumatic Occlusion: Abnormally high differential pressure exceeding normal operating boundaries.