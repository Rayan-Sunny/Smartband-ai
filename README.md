# SmartBand AI

An on-device, wrist-worn activity recognition and fall detection system built on the **Arduino Nano 33 BLE Sense Rev2**. 

Powered by an embedded deep learning model developed with **Edge Impulse**, the system processes 6-axis inertial sensor data directly on the microcontroller with **1 ms inference latency** and streams real-time classifications to a Progressive Web App (PWA) over Bluetooth Low Energy (BLE).

* **Live Web App:** https://Rayan-Sunny.github.io/SmartBand-ai/smartband_ai.html
* **Author:** Rayan Hussain
* **Platform:** Arduino Nano 33 BLE Sense Rev2 (Nordic nRF52840 SoC) + Edge Impulse TinyML

---

## System Architecture

```
[6-Axis IMU (BMI270)]
  │ ax, ay, az, gx, gy, gz @ 16 Hz (10-second sliding window)
  ▼
[Spectral Feature Extraction]
  │ 78 frequency and time-domain features
  ▼
[On-Device Neural Network]
  │ Dense Architecture: 128 → 64 → 32 → 8 output classes
  │ Compiled via Edge Impulse EON™ (1 ms latency, 1.7 KB RAM)
  ▼
[BLE GATT Notification]
  │ Nordic UART Service: "PRED:walking:0.921:waving:0.045"
  ▼
[SmartBand Progressive Web App]
  │ Real-time gauge, confidence tracking, fall alerts, and session history
```

---

## On-Device Performance & Benchmarks

The deep learning model was quantized and compiled using Edge Impulse's EON™ Compiler, optimizing memory and compute constraints specifically for ARM Cortex-M4F microcontrollers.

| Performance Metric | Measured Value |
| :--- | :--- |
| **Inference Latency** | **1 ms** |
| **Peak RAM Consumption** | **1.7 KB** |
| **Flash Memory Footprint** | **35.0 KB** |
| **Validation Accuracy** | **97.7%** |
| **Categorical Loss** | **0.07** |
| **Weighted F1 Score** | **0.98** |

### Validation Confusion Matrix

| Activity | Predicted Accuracy | Class F1 Score |
| :--- | :--- | :--- |
| **Drinking** | 94.7% | 0.95 |
| **Exercise** | 94.4% | 0.97 |
| **Fall** | **100.0%** | **1.00** |
| **Idle** | **100.0%** | **0.97** |
| **Tremor** | 91.7% | 0.96 |
| **Walking** | **100.0%** | **1.00** |
| **Waving** | **100.0%** | **0.98** |
| **Writing** | **100.0%** | **1.00** |

*5 out of 8 classes achieve 100% classification precision, with perfect sensitivity for emergency fall events.*

---

## Recognized Activities

The model classifies 8 distinct physical states from wrist movement dynamics:

| Activity | Category | Motion Profile |
| :--- | :--- | :--- |
| **Walking** | Ambulation | Rhythmic, continuous arm swing during gait cycle |
| **Writing** | Fine Motor | Micro-accelerations with stabilized forearm positioning |
| **Waving** | Gesture | Periodic high-frequency lateral wrist rotation |
| **Drinking** | Functional | Smooth elevation toward mouth followed by controlled descent |
| **Exercise** | Fitness | High-amplitude repetitive physical exertion |
| **Idle** | Baseline | Low-variance resting state without active kinetic input |
| **Tremor** | Neurological | Involuntary oscillating micro-vibrations |
| **Fall** | Safety Event | Sudden impact deceleration followed by cessation of movement |

---

## Machine Learning Pipeline (Edge Impulse)

### 1. Data Collection & Preprocessing
* **Total Recording Duration:** 1 hour, 55 minutes, 14 seconds across 791 labeled samples.
* **Sensor Channels:** 6 axes (`ax, ay, az` in g, `gx, gy, gz` in dps).
* **Sampling Rate:** 16 Hz with zero-padding enabled.
* **Window Configuration:** 10,000 ms window size with a 486.55 ms sliding stride.

### 2. Spectral Analysis (DSP Block)
* Computes FFT-based spectral power distributions, frequency peaks, and standard deviations across all 6 IMU axes.
* Generates an optimized input vector of **78 spectral features**.

### 3. Neural Network Topology
```
┌────────────────────────────────────────────────────────┐
│ Input Layer: 78 Spectral Features                     │
├────────────────────────────────────────────────────────┤
│ Dense Layer (128 neurons, ReLU activation)             │
├────────────────────────────────────────────────────────┤
│ Dense Layer (64 neurons, ReLU activation)              │
├────────────────────────────────────────────────────────┤
│ Dense Layer (32 neurons, ReLU activation)              │
├────────────────────────────────────────────────────────┤
│ Output Layer (8 Softmax probability distributions)    │
└────────────────────────────────────────────────────────┘
```
* **Training Cycles:** 100 epochs
* **Loss Function:** Categorical Cross-Entropy
* **Optimization:** Edge Impulse EON Compiler

---

## Hardware & Firmware Specifications

### Embedded Hardware
* **Microcontroller:** Arduino Nano 33 BLE Sense Rev2
* **SoC:** Nordic Semiconductor nRF52840 (32-bit ARM Cortex-M4F @ 64 MHz, 1 MB Flash, 256 KB RAM)
* **IMU Sensors:** Bosch BMI270 (6-axis accelerometer & gyroscope) + Bosch BMM150 (3-axis magnetometer)

### Arduino Dependencies
Install via the Arduino Library Manager:
* `ArduinoBLE` (Bluetooth Low Energy stack)
* `Arduino_BMI270_BMM150` (Inertial sensor hardware driver)
* `Smart_Wristband_Activity_Recognition_inferencing` (Exported Edge Impulse C++ library)

### BLE GATT Communication Protocol
The microcontroller advertises over the Nordic UART Service (NUS) specification:

| Parameter | Configuration |
| :--- | :--- |
| **Device Name** | `SmartBand` |
| **Service UUID** | `6E400001-B5A3-F393-E0A9-E50E24DCCA9E` |
| **Characteristic UUID** | `6E400003-B5A3-F393-E0A9-E50E24DCCA9E` |
| **Properties** | Read, Notify, Write |
| **Payload Structure** | `PRED:<label1>:<confidence1>:<label2>:<confidence2>` |
| **Sample Payload** | `PRED:walking:0.921:waving:0.045` |

---

## Progressive Web App (PWA) Client

The companion application is built as a single-page, zero-dependency Progressive Web App that communicates directly with the hardware via the **Web Bluetooth API**:

* **Real-Time Telemetry:** Live activity ring, dynamic confidence gauges, and runner-up prediction tracking.
* **Autonomous Fall Alert:** Emergency overlay triggered when a fall is detected, featuring an auditory siren and a 10-second dismiss countdown prior to emergency dispatch.
* **Persistent Session Storage:** Telemetry history survives application restarts using local browser cache (`localStorage`).
* **Data Export:** Session records can be exported directly as CSV or JSON.
* **Offline Operation:** Service worker caching allows the app to be installed to mobile home screens and operated without internet access.

### Browser Compatibility
* **Full Support (Web Bluetooth):** Google Chrome on Android, Google Chrome on macOS / Windows / Linux, Microsoft Edge.
* **Unsupported:** Safari (iOS), Firefox (Web Bluetooth API is not implemented by these engines).

---

## Project Structure

```
SmartBand-ai/
├── smartband_ai.html                    # PWA dashboard & Web Bluetooth client
├── manifest.json                        # PWA installation manifest & icons
├── sw.js                                # Offline service worker
├── README.md                            # Technical documentation
├── model/
│   └── model_info.py                    # Dataset statistics & impulse design metadata
└── nano_ble33_sense_rev2_fusion/
    └── nano_ble33_sense_rev2_fusion.ino # Arduino firmware (IMU capture, inference, BLE)
```

---

## Setup & Flashing

### 1. Flash the Firmware
1. Open `nano_ble33_sense_rev2_fusion/nano_ble33_sense_rev2_fusion.ino` in the Arduino IDE.
2. Select board: **Arduino Nano 33 BLE**.
3. Install the required libraries (`ArduinoBLE`, `Arduino_BMI270_BMM150`, and the Edge Impulse ZIP library).
4. Compile and upload to the board.

### 2. Connect the Web App
1. Open `smartband_ai.html` in Google Chrome (or visit the live GitHub Pages link).
2. Click **Connect to SmartBand**.
3. Select your paired **SmartBand** device from the Web Bluetooth dialog.
4. Wear the device on your wrist and click **Tap to Sample** to begin real-time inferencing.

---

## License

This project is open-source and licensed under the [MIT License](LICENSE).
