# EC-TOF Analyzer

**Prototype Version:** V0.1  
**Platform:** ESP32-S3  
**Firmware:** ESP-IDF + C++  
**Project Status:** Prototype Development

---

## 1. Overview

The **EC-TOF Analyzer** is a portable prototype device designed to combine:

- Electrical conductivity measurement
- Ultrasonic time-of-flight measurement
- Distance calculation
- Timestamped measurement logging
- Local Wi-Fi monitoring
- Local web-based data access
- Battery-powered operation

The system is built around an **ESP32-S3** and uses removable external sensors.

The initial prototype focuses on proving the overall measurement workflow before moving toward production-grade sensing hardware.

---

# 2. Project Goals

The V0.1 prototype must be able to:

1. Measure electrical conductivity of water samples.
2. Measure ultrasonic time of flight.
3. Calculate distance from the measured TOF.
4. Timestamp measurements.
5. Store measurements on MicroSD.
6. Display measurements on a local TFT display.
7. Allow physical control using buttons and a rotary encoder.
8. Provide a local Wi-Fi interface.
9. Provide a browser-based dashboard.
10. Support EC and TOF calibration.
11. Operate from a rechargeable battery.

---

# 3. Important Prototype Limitation

The **HC-SR04 is a prototype and feasibility component only**.

It is an air ultrasonic ranging module.

It must **not** be considered the final laboratory-grade ultrasonic measurement solution.

The firmware therefore implements TOF through a replaceable sensor abstraction.

This allows the HC-SR04 to be replaced later with a more appropriate ultrasonic transducer, receiver, analog front-end, or dedicated TOF hardware without redesigning the entire application.

---

# 4. Hardware

## Main Controller

**ESP32-S3-DevKitC-1-N8R8**

Responsibilities:

- Application control
- Sensor communication
- Measurement processing
- Display control
- Data logging
- Wi-Fi
- User interface
- System monitoring

---

## Electrical Conductivity Sensor

**DFRobot SEN0707 Industrial Water Conductivity Sensor**

Interface:

```text
RS485
Modbus RTU
```

Key characteristics:

- K = 10
- Conductivity range: 10 to 20,000 µS/cm
- Resolution: 1 µS/cm
- Accuracy: ±1% FS
- Supply: 10 to 30 V DC
- IP68
- Built-in temperature compensation

The sensor requires a dedicated higher-voltage supply.

---

## RS485 Interface

**MAX3485**

Architecture:

```text
ESP32-S3
    |
   UART
    |
MAX3485
    |
  RS485
    |
SEN0707
```

---

## Ultrasonic Sensor

**HC-SR04**

Purpose:

- V0.1 TOF feasibility testing
- Air-distance prototype testing
- Timing subsystem development

The ECHO output must be level-shifted before connecting to an ESP32-S3 GPIO.

---

## Display

**4" 480x320 ST7796 SPI TFT**

Purpose:

- Measurement display
- Device status
- Calibration
- Configuration
- System information

The display is non-touch.

---

## Real-Time Clock

**DS3231**

Purpose:

- Measurement timestamps
- Offline timekeeping
- Data logging

The RTC communicates through I2C.

I2C is intentionally limited to internal hardware in V0.1.

---

## Storage

**MicroSD**

Purpose:

- Measurement logging
- CSV export
- System logs

Example structure:

```text
/ECTOF/
├── config/
├── data/
│   └── YYYY/
│       └── MM/
│           └── DD.csv
└── logs/
```

---

## Physical Controls

The device uses:

- Rotary encoder
- Encoder push button
- START button
- BACK button
- Power switch

---

## Battery

Target:

**3.7 V 5000 mAh Li-ion/LiPo battery**

Power architecture:

```text
3.7 V Battery
      |
      +---------------------+
      |                     |
      v                     v
  12 V Boost           5 V Regulator
      |                     |
      v                     v
 SEN0707              TFT / HC-SR04
                            |
                            v
                       3.3 V Logic
                            |
                            v
                         ESP32-S3
```

USB-C is used for charging.

Actual battery runtime must be measured during prototype validation.

---

# 5. System Architecture

```text
                    +----------------------+
                    |      User Layer      |
                    | TFT + Buttons + Web  |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |  Application Layer   |
                    | Measurement Manager  |
                    | Calibration Manager  |
                    | Configuration        |
                    +----------+-----------+
                               |
              +----------------+----------------+
              |                                 |
              v                                 v
     +------------------+              +------------------+
     | Sensor Managers  |              | Storage / RTC    |
     | EC / TOF         |              | SD / DS3231      |
     +--------+---------+              +------------------+
              |
              v
     +---------------------------+
     |       Driver Layer        |
     | UART / RS485 / SPI / GPIO |
     | I2C / Timer               |
     +-------------+-------------+
                   |
                   v
     +---------------------------+
     |         Hardware          |
     +---------------------------+
```

The system follows a layered architecture.

Hardware drivers must not contain UI logic.

UI components must not directly control low-level hardware.

---

# 6. Measurement Flow

The primary measurement sequence is:

```text
IDLE
  ↓
START
  ↓
TOF_TRIGGER
  ↓
TOF_WAIT
  ↓
TOF_PROCESS
  ↓
EC_READ
  ↓
VALIDATE
  ↓
DISPLAY
  ↓
LOG
  ↓
READY
```

If a measurement fails:

```text
Measurement
     ↓
   ERROR
     ↓
Display Error
     ↓
   READY
```

The system must never log a failed measurement as `VALID`.

---

# 7. Measurement Data

The primary measurement structure contains:

```text
Measurement ID
Timestamp
Conductivity
TOF
Distance
Status
```

Example:

```text
ID: 000123
Timestamp: 2026-10-08T15:32:10
Conductivity: 4820 µS/cm
TOF: 12.482 µs
Distance: 18.73 mm
Status: VALID
```

---

# 8. Data Logging

Measurements are stored as CSV.

Format:

```csv
timestamp,conductivity,tof_us,distance_mm,status
```

Example:

```csv
2026-10-08T15:32:10,4820,12.482,18.73,VALID
```

The CSV format is intentionally simple so the data can be opened using:

- Microsoft Excel
- Google Sheets
- LibreOffice
- Python
- MATLAB
- Other data-analysis tools

---

# 9. User Interface

## Main Screen

The main screen displays:

```text
+------------------------------------------+
|             EC-TOF ANALYZER              |
+------------------------------------------+
|                                          |
| Conductivity                             |
| 4.82 mS/cm                               |
|                                          |
| Ultrasonic TOF                           |
| 12.482 us                                |
|                                          |
| Distance                                 |
| 18.73 mm                                 |
|                                          |
| Status: READY                            |
+------------------------------------------+
```

Optional system information may include:

- Battery level
- SD status
- Wi-Fi status

---

# 10. UI Screens

The V0.1 firmware includes the following logical screens:

```text
Main
Measurements
Calibration
Configuration
System Status
```

Navigation:

```text
Encoder CW/CCW
        ↓
Select

Encoder Press
        ↓
Open

BACK
        ↓
Return

START
        ↓
Start Measurement
```

---

# 11. Local Web Interface

The ESP32-S3 operates as a local Wi-Fi access point.

Default prototype configuration:

```text
SSID:
EC-TOF-Analyzer

Example IP:
192.168.4.1
```

The user can connect using a phone, tablet, or computer.

---

## Web Pages

```text
/dashboard
/measurements
/calibration
/configuration
/system
```

---

## REST API

### Device

```http
GET /api/device
```

### System

```http
GET /api/system/status
```

### Current Measurement

```http
GET /api/measurement/current
```

### Start Measurement

```http
POST /api/measurement/start
```

### Measurement History

```http
GET /api/measurements
```

### Export Data

```http
GET /api/export/YYYY-MM-DD.csv
```

### EC Calibration

```http
GET  /api/calibration/ec
POST /api/calibration/ec
```

### Configuration

```http
GET  /api/config
POST /api/config
```

The web interface communicates with the application layer.

It must not directly access sensor drivers.

---

# 12. Calibration

## EC Calibration

The SEN0707 uses the manufacturer's recommended calibration process.

V0.1 supports:

- Reading the current conductivity
- Applying calibration values
- Saving calibration configuration
- Validating the result

The supplied calibration solution should be used according to the sensor manufacturer's procedure.

---

## TOF Calibration

TOF calibration uses a known physical distance.

Basic process:

```text
Known Distance
      ↓
Measure TOF
      ↓
Calculate Calibration
      ↓
Store Calibration
      ↓
Validate
```

Calibration values are stored in ESP32-S3 NVS.

---

# 13. Configuration

Persistent configuration is stored using ESP-IDF NVS.

Potential settings include:

```text
Device Name
Sensor Address
Measurement Timeout
TOF Calibration
EC Calibration
Display Settings
Wi-Fi Settings
Logging Settings
```

Configuration must survive reboot.

---

# 14. Error Handling

The system must detect and handle:

```text
EC sensor disconnected
EC communication timeout
RS485 communication error
TOF timeout
TOF invalid measurement
SD card unavailable
SD write failure
RTC failure
Wi-Fi initialization failure
Configuration error
Invalid calibration
Low battery
```

Errors must be:

1. Detected.
2. Reported.
3. Logged when possible.
4. Displayed to the user.
5. Recovered automatically when safe.

---

# 15. Software Structure

The ESP-IDF project follows this structure:

```text
ec-tof-analyzer/
│
├── CMakeLists.txt
├── sdkconfig
├── sdkconfig.defaults
├── README.md
│
├── main/
│   ├── main.cpp
│   │
│   ├── app/
│   ├── common/
│   │
│   ├── sensors/
│   │   ├── ec/
│   │   └── tof/
│   │
│   ├── drivers/
│   │   ├── gpio/
│   │   ├── i2c/
│   │   ├── spi/
│   │   ├── uart/
│   │   ├── rs485/
│   │   └── timer/
│   │
│   ├── display/
│   ├── input/
│   ├── rtc/
│   ├── storage/
│   ├── web/
│   ├── power/
│   └── system/
│
└── components/
```

---

# 16. Development Environment

Recommended development environment:

- Visual Studio Code
- ESP-IDF extension
- ESP-IDF
- Git
- Serial terminal
- USB-C connection
- ESP32-S3 development board

The firmware uses:

```text
Language:
C++

Framework:
ESP-IDF

Target:
ESP32-S3
```

---

# 17. Build and Flash

Typical ESP-IDF workflow:

```bash
idf.py set-target esp32s3
idf.py build
idf.py flash
idf.py monitor
```

For a clean rebuild:

```bash
idf.py fullclean
idf.py build
```

The exact commands may vary depending on the development environment.

---

# 18. Development Sequence

Implementation should follow this order:

```text
1. ESP-IDF Project
2. ESP32-S3 Bring-Up
3. GPIO
4. TFT Driver
5. Input Manager
6. RTC
7. MicroSD
8. UART
9. RS485
10. SEN0707 Modbus Driver
11. EC Measurement
12. HC-SR04 Driver
13. TOF Measurement
14. Measurement Manager
15. Calibration
16. Configuration
17. Integrated UI
18. Wi-Fi AP
19. REST API
20. Web UI
21. Error Handling
22. Battery System
23. Enclosure
24. System Testing
25. V0.1 Validation
```

Each subsystem should be tested before integration.

---

# 19. Documentation

Project documentation is divided into the following documents:

| Document | Description |
|---|---|
| `01-product-requirements.md` | Product goals and functional requirements |
| `02-technical-requirements.md` | Technical requirements and constraints |
| `03-system-architecture.md` | Overall system architecture |
| `04-hardware-design.md` | Hardware design and component architecture |
| `05-software-design.md` | Firmware architecture and software design |
| `06-implementation-plan.md` | Development phases and implementation roadmap |
| `README.md` | Project overview and developer entry point |

---

# 20. Testing Strategy

Testing occurs at multiple levels.

## Driver Testing

Test:

- GPIO
- SPI
- I2C
- UART
- RS485
- SD
- RTC
- Sensors

## Integration Testing

Test:

- EC + RS485
- TOF + timer
- Measurement Manager
- Display + application
- SD + RTC
- Wi-Fi + application

## System Testing

Test:

- Complete measurement cycle
- Sensor failures
- SD failures
- Reboot recovery
- Battery operation
- Long-duration operation

---

# 21. V0.1 Acceptance Criteria

The V0.1 prototype must:

### Measurement

- Measure conductivity.
- Measure ultrasonic TOF.
- Calculate prototype distance.
- Timestamp measurements.
- Validate measurements.

### Display

- Show measurement results.
- Show system status.
- Show errors.
- Support physical navigation.

### Storage

- Store measurements on MicroSD.
- Use CSV format.
- Preserve data after reboot.

### Calibration

- Support EC calibration.
- Support TOF calibration.
- Persist calibration values.

### Connectivity

- Provide local Wi-Fi.
- Provide browser dashboard.
- Provide measurement history.
- Provide CSV export.

### Power

- Operate from battery.
- Support USB-C charging.
- Provide physical power control.

### Reliability

- Detect sensor failures.
- Detect communication timeouts.
- Handle SD failures.
- Avoid system hangs during normal errors.

---

# 22. Known Limitations

V0.1 is not a production instrument.

Known limitations include:

### Ultrasonic Hardware

The HC-SR04 is not the final TOF measurement hardware.

### Accuracy

V0.1 accuracy must be established through testing.

No production accuracy specification should be claimed before validation.

### Battery

Battery runtime must be measured on the physical prototype.

### Power Electronics

The development-board and module-based power architecture is suitable for prototyping but may require redesign for production.

### Enclosure

The first enclosure is expected to be a prototype enclosure.

### PCB

V0.1 may use development boards and modules.

A custom PCB should be considered after the prototype architecture is validated.

---

# 23. Prototype Philosophy

The project follows these principles:

### Prototype First

Use readily available components to validate the concept.

### Modular Hardware

Sensors must be removable.

### Modular Firmware

Sensor implementations must be replaceable.

### No Unnecessary Complexity

Do not build production-level infrastructure before the measurement concept is proven.

### Fail Safely

Invalid measurements must not be presented as valid.

### Test Before Integrating

Every subsystem should be tested independently.

### Design for V1

The architecture should allow future improvements without requiring a complete rewrite.

---

# 24. Future Development

Potential future improvements include:

## Ultrasonic

- Laboratory-grade ultrasonic transducer
- Dedicated receiver
- Analog front-end
- Better timing hardware
- Signal processing
- Liquid/acoustic TOF implementation

## Hardware

- Custom PCB
- Improved power management
- Battery fuel gauge
- ESD protection
- EMI filtering
- Industrial connectors
- Improved enclosure

## Firmware

- Advanced TOF processing
- Signal quality metrics
- Measurement statistics
- Automatic calibration
- Configuration backup
- Firmware update system
- Improved diagnostics

## Connectivity

- USB data interface
- Bluetooth
- Wi-Fi network mode
- Remote monitoring
- Cloud integration

These features are outside V0.1.

---

# 25. Project Status

Current status:

```text
Project Definition       [COMPLETE]
Product Requirements     [COMPLETE]
Technical Requirements   [COMPLETE]
System Architecture      [COMPLETE]
Hardware Design          [COMPLETE]
Software Design          [COMPLETE]
Implementation Plan      [COMPLETE]

Firmware Development     [NEXT]
Hardware Assembly        [NEXT]
Prototype Validation     [PENDING]
```

---

# 26. Quick Start for Developers

A new developer should read the documentation in this order:

```text
README.md
    ↓
01-product-requirements.md
    ↓
02-technical-requirements.md
    ↓
03-system-architecture.md
    ↓
04-hardware-design.md
    ↓
05-software-design.md
    ↓
06-implementation-plan.md
```

Then begin implementation with:

```text
ESP-IDF Project
      ↓
ESP32-S3 Bring-Up
      ↓
Display
      ↓
Input
      ↓
RTC
      ↓
SD
      ↓
RS485
      ↓
EC
      ↓
TOF
```

---

# 27. Repository Goal

The repository should contain everything required to reproduce the V0.1 prototype:

```text
Firmware
Hardware documentation
Pin configuration
Configuration files
Calibration information
Test procedures
Measurement examples
Development documentation
```

Hardware-specific configuration must be documented rather than embedded as unexplained constants.

---

# 28. Final System Workflow

The complete V0.1 workflow is:

```text
             POWER ON
                 |
                 v
        Initialize System
                 |
                 v
          Hardware Check
                 |
                 v
               READY
                 |
                 v
          User Presses START
                 |
                 v
        Ultrasonic Measurement
                 |
                 v
             TOF Result
                 |
                 v
         Prototype Distance
                 |
                 v
         EC Sensor Reading
                 |
                 v
        Measurement Validation
                 |
           +-----+-----+
           |           |
         VALID        ERROR
           |           |
           v           v
        Display     Show Error
           |           |
           v           |
        Timestamp      |
           |           |
           v           |
        Save CSV       |
           |           |
           +-----+-----+
                 |
                 v
               READY
```

---

# 29. Final V0.1 Hardware Baseline

| Component | Selected Hardware |
|---|---|
| MCU | ESP32-S3-DevKitC-1-N8R8 |
| EC Sensor | DFRobot SEN0707 |
| EC Interface | RS485 Modbus RTU |
| RS485 | MAX3485 |
| TOF Sensor | HC-SR04 |
| Display | 4" 480x320 ST7796 SPI TFT |
| RTC | DS3231 |
| Storage | MicroSD |
| Input | Rotary Encoder + START + BACK |
| Battery | 3.7 V 5000 mAh |
| EC Supply | 3.7 V → 12 V Boost |
| Display/TOF Supply | 3.7 V → 5 V Regulation |
| Logic | 3.3 V |
| Charging | USB-C |
| Enclosure | Prototype enclosure |
| External Sensors | Removable |

---

# 30. Final V0.1 Software Baseline

| Area | Implementation |
|---|---|
| Framework | ESP-IDF |
| Language | C++ |
| Architecture | Layered / Modular |
| Sensor API | Abstract interfaces |
| EC Protocol | Modbus RTU |
| Display | SPI |
| RTC | I2C |
| Storage | SPI MicroSD |
| Input | GPIO |
| TOF Timing | Microsecond timer |
| Configuration | NVS |
| Logging | CSV |
| Wireless | ESP32 Wi-Fi AP |
| Web Server | Local HTTP |
| API | REST |
| Calibration | EC + TOF |
| Error Handling | Centralized |
| Firmware Update | USB for V0.1 |

---

# 31. Engineering Principle

The most important design decision in V0.1 is:

> **Prove the measurement concept before optimizing the hardware.**

The EC subsystem should be developed independently from the TOF subsystem.

The TOF subsystem must remain replaceable.

The application layer must operate on measurement interfaces rather than specific hardware.

This allows the project to evolve from a development-board prototype into a production-oriented instrument without requiring a complete firmware rewrite.

---

# 32. Project Definition of Done

The EC-TOF Analyzer V0.1 project is complete when:

```text
[ ] Hardware assembled
[ ] ESP32-S3 firmware builds
[ ] Display operational
[ ] Physical controls operational
[ ] RTC operational
[ ] MicroSD operational
[ ] RS485 operational
[ ] SEN0707 operational
[ ] EC measurement operational
[ ] HC-SR04 operational
[ ] TOF measurement operational
[ ] Measurement Manager operational
[ ] Calibration operational
[ ] Configuration persistence operational
[ ] Local Wi-Fi operational
[ ] Web dashboard operational
[ ] CSV export operational
[ ] Battery operation validated
[ ] Error handling validated
[ ] Integrated testing completed
[ ] Measurement repeatability tested
[ ] Prototype enclosure assembled
[ ] V0.1 documentation completed
```

---

## EC-TOF Analyzer V0.1

**Build the prototype. Validate the measurement concept. Measure the limitations. Then design the production system.**