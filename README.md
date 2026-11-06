# ESP32 Access Control & Multi-Relay Traffic Switching Gateway

[![Platform](https://img.shields.io/badge/Platform-PlatformIO-orange.svg)](https://platformio.org/)
[![Hardware](https://img.shields.io/badge/Hardware-ESP32--DevKit-blue.svg)](https://www.espressif.com/)
[![Sensor](https://img.shields.io/badge/RFID-MFRC522%20SPI-lightblue.svg)](https://www.nxp.com/)
[![Database](https://img.shields.io/badge/Database-Firebase%20RTDB-yellow.svg)](https://firebase.google.com/)

An integrated IoT access control and automation gateway built on the ESP32. The system combines high-frequency RC522 RFID card authentication, multi-relay and dual-phase status LED switching, inter-board hardware UART communication, and Google Firebase Realtime Database cloud telemetry.

---

## System Architecture

```
                                +-------------------+
                                |  RC522 RFID (SPI) |
                                +---------+---------+
                                          |
                                          v
+-------------------+           +-------------------+           +--------------------+
| Traffic Controller| <=======> |   ESP32 Gateway   | <=======> | Google Firebase    |
| (UART 115200 baud)|           |   (This Device)   |           | Realtime Database  |
+-------------------+           +---------+---------+           +--------------------+
                                          |
                        +-----------------+-----------------+
                        |                 |                 |
                        v                 v                 v
                 +--------------+  +--------------+  +--------------+
                 | Relay Output |  | Dual Red/Grn |  | Manual Inputs|
                 | (GPIO 11)    |  | Signal LEDs  |  | (SW1 & SW2)  |
                 +--------------+  +--------------+  +--------------+
```

---

## Features

- **MFRC522 RFID Security Gate**: Scans 13.56MHz cards via hardware SPI. Authorizes registered UIDs (e.g. `37 E0 96 04`), unlocks actuator relays, and signals the upstream controller to enter `"Manual"` mode.
- **Relay & Multi-Phase Traffic Light Driving**:
  - Relay control on `GPIO 11`.
  - Phase 1 Signals: Red (`GPIO 12`), Green (`GPIO 15`).
  - Phase 2 Signals: Red (`GPIO 13`), Green (`GPIO 17`).
  - Manual toggle inputs (`SW1` on GPIO 14, `SW2` on GPIO 16).
- **Inter-Controller UART Bridge**: Communicates over `Serial1` (GPIO 33 RX / GPIO 32 TX at 115200 baud) to exchange traffic densities (`R1`, `R1T`, `R2`, `R2T`, `Mode`).
- **Cloud Synchronization & Emergency Trigger**:
  - Periodically uploads telemetry to Firebase (`/R1C`, `/R1T`, `/R2C`, `/R2T`, `/Mode`).
  - Monitors `/Set == 1` to immediately dispatch emergency signals to local controllers.

---

## Pinout Matrix

### 1. RC522 RFID Reader (SPI)

| RC522 Pin | ESP32 GPIO | Description |
| :--- | :--- | :--- |
| **VCC** | `3.3V` | **3.3V Power ONLY** |
| **RST** | `GPIO 4` | Reset Pin |
| **SDA (SS)** | `GPIO 5` | SPI Chip Select |
| **SCK** | `GPIO 18` | SPI Clock |
| **MISO** | `GPIO 19` | Master In Slave Out |
| **MOSI** | `GPIO 23` | Master Out Slave In |
| **GND** | `GND` | Common Ground |

### 2. Actuators, Switches & Status LEDs

| Component | ESP32 GPIO | Pin Type | Notes |
| :--- | :--- | :--- | :--- |
| **Main Relay** | `GPIO 11` | Output | Actuator / Barrier relay |
| **Switch 1 (SW1)** | `GPIO 14` | Input | Manual mode toggle for Phase 1 |
| **Switch 2 (SW2)** | `GPIO 16` | Input | Manual mode toggle for Phase 2 |
| **Red LED 1** | `GPIO 12` | Output | Phase 1 Stop signal |
| **Green LED 1**| `GPIO 15` | Output | Phase 1 Go signal |
| **Red LED 2** | `GPIO 13` | Output | Phase 2 Stop signal |
| **Green LED 2**| `GPIO 17` | Output | Phase 2 Go signal |

### 3. UART Serial1 Interface

| Pin | Direction | Connected Device |
| :--- | :--- | :--- |
| `GPIO 33` | `RX1` (In) | Upstream Microcontroller (TX) |
| `GPIO 32` | `TX1` (Out)| Upstream Microcontroller (RX) |

---

## Software Configuration

Configured in [`platformio.ini`](platformio.ini):

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
lib_deps = 
    mobizt/Firebase ESP32 Client@^4.4.17
    miguelbalboa/MFRC522@^1.4.12
```

---

## Getting Started

```bash
# Build
pio run

# Flash to ESP32
pio run --target upload

# Serial monitor
pio device monitor -b 115200
```

---

## Project Structure

```
espreciever3/
├── src/
│   └── main.cpp         # RFID validation, relay toggling, UART bridge & Firebase sync
└── platformio.ini       # PlatformIO configuration
```

---

## License

MIT License.
