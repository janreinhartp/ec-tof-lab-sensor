# EC-TOF Analyzer V0.1
# Product Requirements Document

## 1. Product Overview

The **EC-TOF Analyzer** is a portable measurement device designed to combine:

- Electrical conductivity measurement.
- Ultrasonic time-of-flight measurement.
- Distance calculation.
- Timestamped measurement logging.
- Local display.
- Local Wi-Fi access.
- Rechargeable battery operation.

The V0.1 version is a functional prototype.

The primary goal is to validate the complete measurement workflow and establish a hardware and software architecture that can be upgraded to a production-grade ultrasonic measurement system later.

---

# 2. Product Objective

The primary objective of V0.1 is to create a portable analyzer that can:

1. Measure the electrical conductivity of a liquid sample.
2. Perform an ultrasonic time-of-flight measurement.
3. Calculate distance from the measured TOF.
4. Display measurement results locally.
5. Record measurement results with timestamps.
6. Allow measurements to be initiated using physical controls.
7. Provide a local Wi-Fi interface.
8. Provide a simple battery level indicator.
9. Operate without an internet connection.
10. Use removable external sensors.

---

# 3. V0.1 Product Scope

The V0.1 prototype includes:

| Feature | V0.1 |
|---|---|
| Electrical conductivity | Yes |
| Ultrasonic TOF | Yes |
| Distance calculation | Yes |
| Local TFT display | Yes |
| Physical controls | Yes |
| Measurement logging | Yes |
| RTC timestamping | Yes |
| MicroSD storage | Yes |
| Local Wi-Fi | Yes |
| REST API | Yes |
| Web dashboard | Yes |
| Rechargeable battery | Yes |
| Battery voltage measurement | Yes |
| Simple battery percentage | Yes |
| Battery current measurement | No |
| Battery power measurement | No |
| Cloud connectivity | No |
| Touchscreen | No |
| Web OTA | No |
| Production-grade ultrasonic system | No |

---

# 4. Target User

The initial system is intended for technical and engineering users who need a portable measurement instrument for prototype testing and laboratory-style evaluation.

The device should prioritize:

- Measurement reliability.
- Clear results.
- Repeatability.
- Simple operation.
- Easy data retrieval.
- Replaceable sensors.
- Maintainability.

The V0.1 interface does not need to be consumer-oriented.

---

# 5. Core User Workflow

The primary user workflow is:

```text
Power ON
   │
   ▼
System Initialization
   │
   ▼
READY
   │
   ▼
Prepare Sample
   │
   ▼
Position Sensors
   │
   ▼
Press START
   │
   ▼
Ultrasonic TOF Measurement
   │
   ▼
Distance Calculation
   │
   ▼
Conductivity Measurement
   │
   ▼
Measurement Validation
   │
   ├────────────── Invalid ──────► ERROR
   │
   ▼
Display Result
   │
   ▼
Timestamp Result
   │
   ▼
Save to MicroSD
   │
   ▼
Update Web Interface
   │
   ▼
READY
```

---

# 6. Electrical Conductivity Requirements

## 6.1 Sensor

V0.1 shall use the:

**DFRobot SEN0707 Industrial Water Conductivity Sensor**

The sensor communicates through RS485 using Modbus-RTU.

### Target specifications

| Parameter | Requirement |
|---|---|
| Cell constant | K=10 |
| Range | 10 to 20,000 µS/cm |
| Resolution | 1 µS/cm |
| Accuracy | ±1% FS |
| Interface | RS485 |
| Protocol | Modbus-RTU |
| Supply | 10 to 30 V DC |
| Protection | IP68 |

The actual usable measurement range must be confirmed against the customer's sample requirements.

---

# 7. Conductivity Measurement Requirements

The system shall:

- Communicate with the SEN0707 through RS485.
- Use Modbus-RTU.
- Detect communication timeouts.
- Detect invalid Modbus responses.
- Detect CRC errors.
- Detect sensor disconnection where possible.
- Read conductivity values.
- Validate conductivity values.
- Display conductivity results.
- Log conductivity results.
- Include conductivity in the web API.

The firmware shall isolate the SEN0707 implementation behind an EC sensor interface.

---

# 8. Ultrasonic TOF Requirements

## 8.1 V0.1 Sensor

V0.1 shall use an **HC-SR04** for initial ultrasonic feasibility testing.

The HC-SR04 is a prototype component.

It is not considered the final ultrasonic measurement solution.

---

# 9. Ultrasonic Measurement Objective

The V0.1 ultrasonic subsystem shall demonstrate:

- Trigger generation.
- Echo detection.
- Microsecond-level timing.
- TOF calculation.
- Distance calculation.
- Measurement timeout handling.
- Measurement validation.
- Display of TOF.
- Display of calculated distance.
- Logging of TOF and distance.

---

# 10. Ultrasonic Limitation

The HC-SR04 is an air ultrasonic ranging sensor.

Therefore:

**V0.1 shall not be considered a production-grade liquid ultrasonic measurement instrument.**

The purpose of the V0.1 ultrasonic subsystem is to validate:

- The measurement workflow.
- Timing architecture.
- Software abstraction.
- Data processing.
- User interface.
- Logging.
- Overall system integration.

A future production version may use a dedicated ultrasonic transducer, receiver, amplifier, and signal-processing system.

---

# 11. Distance Calculation

For the V0.1 air prototype:

```text
Distance = TOF × Speed of Sound / 2
```

The speed of sound shall be configurable.

The calculation must be isolated from the HC-SR04 driver so that the future acoustic measurement model can be replaced.

---

# 12. Sensor Replaceability

External sensors must be removable.

The system shall not hardwire the EC sensor or ultrasonic sensor directly to the main PCB.

The architecture shall allow:

```text
Current:

ITofSensor
    └── HC-SR04

Future:

ITofSensor
    ├── HC-SR04
    ├── Laboratory Ultrasonic Sensor
    └── Custom Acoustic System
```

The same principle applies to the conductivity sensor.

---

# 13. Local Display Requirements

The device shall include a:

**4-inch 480x320 ST7796 SPI TFT**

The display shall show:

- Conductivity.
- Ultrasonic TOF.
- Distance.
- Measurement status.
- Battery percentage.
- System warnings.
- Basic system status.

Example:

```text
┌──────────────────────────────────────┐
│ EC-TOF ANALYZER               🔋 78% │
│                                      │
│ Conductivity                         │
│ 4.82 mS/cm                            │
│                                      │
│ Ultrasonic TOF                       │
│ 12.482 us                             │
│                                      │
│ Distance                             │
│ 18.73 mm                              │
│                                      │
│ Status: READY                         │
└──────────────────────────────────────┘
```

The display shall not require a touchscreen.

---

# 14. Physical Control Requirements

The device shall provide:

### Rotary Encoder

Used for:

- Menu navigation.
- Parameter selection.
- Value adjustment.
- Selection confirmation.

### START Button

Used to:

- Start a measurement.

### BACK Button

Used to:

- Return to the previous screen.
- Cancel supported operations.
- Exit configuration screens.

---

# 15. Measurement Logging

The device shall log measurement results to MicroSD.

Each valid measurement should contain:

```text
timestamp
conductivity
tof_us
distance_mm
status
```

Example:

```csv
timestamp,conductivity,tof_us,distance_mm,status
2026-10-08T15:32:10,4820,12.482,18.73,VALID
```

---

# 16. Data Storage

The recommended storage structure is:

```text
/ECTOF/
    config/
    data/
        YYYY/
            MM/
                DD.csv
    logs/
```

The system shall create required directories automatically.

A missing or failed SD card shall not crash the measurement application.

The user shall receive a warning when measurement logging is unavailable.

---

# 17. Timestamp Requirements

The device shall use a DS3231 RTC.

The RTC shall provide timestamps without requiring internet access.

Timestamp format:

```text
YYYY-MM-DDTHH:MM:SS
```

Example:

```text
2026-10-08T15:32:10
```

---

# 18. Battery Requirements

The device shall be battery powered.

Target battery:

```text
3.7 V
~5000 mAh
Rechargeable Li-ion
```

The battery system shall include appropriate protection.

---

# 19. Battery Charging

The device shall support USB-C charging.

The charging system shall be compatible with the selected single-cell Li-ion battery.

The USB-C connector should be accessible from outside the enclosure.

---

# 20. Battery Indicator

V0.1 shall provide a simple battery indicator.

The system shall:

- Measure battery voltage.
- Estimate battery percentage.
- Display battery percentage.
- Display low-battery status.
- Display critical-battery status.

The battery indicator is an approximate user-facing estimate.

It is not intended to provide precision state-of-charge measurement.

---

# 21. Battery Measurement Exclusions

V0.1 shall **not** measure:

- Battery current.
- Battery power.
- Instantaneous system power.
- Energy consumption.
- Detailed battery charge/discharge characteristics.

V0.1 shall not include an INA226 or dedicated battery fuel gauge.

---

# 22. Power Architecture

The system shall provide the voltage rails required by the selected hardware.

Expected architecture:

```text
3.7 V Battery
     │
     ├──► 12 V Boost ──► SEN0707
     │
     ├──► 5 V Rail ─────► TFT / HC-SR04 where required
     │
     └──► 3.3 V Rail ───► ESP32-S3 / Logic
```

The exact regulator selection shall be finalized during hardware implementation.

---

# 23. Local Wi-Fi Requirements

The ESP32-S3 shall provide a local Wi-Fi access point.

Example:

```text
SSID:
EC-TOF-Analyzer

IP:
192.168.4.1
```

The device shall not require an internet connection for normal operation.

---

# 24. Web Interface

The local web interface shall provide:

### Dashboard

- Current conductivity.
- Current TOF.
- Distance.
- Battery level.
- System status.
- Last measurement.

### Measurements

- Measurement history.
- Timestamp.
- Measurement values.
- Status.

### Calibration

- EC calibration.
- TOF calibration.

### Configuration

- Measurement settings.
- Sensor settings.
- Device settings.

### System Status

- Firmware version.
- Sensor status.
- RTC status.
- SD status.
- Wi-Fi status.
- Battery status.

---

# 25. REST API Requirements

The device shall expose:

```text
GET  /api/device

GET  /api/system/status

GET  /api/measurement/current

POST /api/measurement/start

GET  /api/measurements

GET  /api/export/YYYY-MM-DD.csv

GET  /api/calibration/ec

POST /api/calibration/ec

GET  /api/config

POST /api/config
```

The API shall use JSON where appropriate.

The API shall not directly access sensor drivers.

---

# 26. Measurement State Requirements

The measurement workflow shall use a defined state machine.

Baseline states:

```text
IDLE
START
TOF_TRIGGER
TOF_WAIT
TOF_PROCESS
EC_READ
VALIDATE
DISPLAY
LOG
READY
```

Error states shall handle:

- TOF timeout.
- TOF sensor failure.
- EC timeout.
- EC communication failure.
- Invalid measurement.
- SD logging failure.

---

# 27. Measurement Validation

A measurement shall be considered valid only when:

- TOF measurement completed.
- TOF is within configured limits.
- Distance is within configured limits.
- EC sensor responded successfully.
- Conductivity is within configured limits.
- Timestamp is available.

Invalid measurements shall not be presented as valid results.

---

# 28. Error Handling Requirements

The system shall detect and report:

### Conductivity errors

- Sensor timeout.
- RS485 communication error.
- Modbus CRC error.
- Invalid response.
- Sensor disconnection.

### TOF errors

- No echo.
- Timeout.
- Invalid TOF.
- Invalid distance.
- Sensor disconnection.

### Storage errors

- SD card missing.
- SD initialization failure.
- File creation failure.
- Write failure.

### RTC errors

- RTC unavailable.
- Invalid time.
- Communication failure.

### Battery errors

- Low battery.
- Critical battery.
- Battery ADC failure.

---

# 29. Configuration Requirements

The following parameters shall be configurable where applicable:

- EC Modbus slave ID.
- Measurement limits.
- TOF limits.
- Speed of sound.
- Measurement timeout.
- Calibration parameters.
- Wi-Fi settings.
- Device settings.

Configuration shall persist across reboot.

Configuration shall be stored using ESP-IDF NVS.

---

# 30. Calibration Requirements

The device shall support:

## EC Calibration

Calibration shall follow the SEN0707 manufacturer's procedure.

The supplied conductivity calibration solution may be used for verification.

## TOF Calibration

TOF shall support calibration against a known reference distance.

The calibration system must remain replaceable for future ultrasonic hardware.

---

# 31. Firmware Requirements

Firmware shall use:

- ESP-IDF.
- C++.
- FreeRTOS through ESP-IDF.
- LVGL for the TFT UI.
- ESP-IDF HTTP Server.
- ESP-IDF NVS.
- ESP-IDF logging.

The firmware shall use modular components.

---

# 32. Software Architecture Requirements

The software shall separate:

```text
Application
    │
    ▼
Services
    │
    ▼
Drivers
    │
    ▼
Hardware
```

The architecture shall include abstraction layers for:

- EC sensors.
- TOF sensors.
- Storage.
- RTC.
- Display.
- Input.
- Power.
- Wi-Fi.

---

# 33. Reliability Requirements

The device shall:

- Recover from temporary sensor communication failures.
- Detect sensor timeouts.
- Avoid indefinite blocking.
- Use watchdog protection.
- Handle SD card failure without crashing.
- Handle Wi-Fi client disconnects.
- Handle low battery safely.
- Return to a known state after a failed measurement.

---

# 34. Data Integrity Requirements

Measurement records shall:

- Include timestamps.
- Use a consistent CSV format.
- Include measurement status.
- Avoid partial records where practical.
- Be written using controlled file operations.

The system should minimize the possibility of corrupting previously recorded measurements.

---

# 35. User Interface Requirements

The UI should prioritize:

1. Current measurement.
2. Measurement status.
3. Battery state.
4. Sensor errors.
5. Simple navigation.

The user should be able to start a measurement without navigating through multiple menus.

The main measurement screen should remain uncluttered.

---

# 36. Safety Requirements

The hardware and software shall consider:

- Li-ion battery safety.
- Battery over-discharge.
- Short-circuit protection.
- Protected charging.
- Safe external connectors.
- Correct voltage levels.
- ESP32 GPIO protection.
- HC-SR04 ECHO level shifting.
- Secure battery mounting.
- Electrical isolation where required.

---

# 37. Environmental Requirements

The V0.1 enclosure should protect the electronics from normal handling and laboratory use.

The external EC sensor is IP68-rated according to its specification.

The overall analyzer enclosure is not considered IP-rated unless specifically designed and tested for an enclosure rating.

The device should not be treated as a certified industrial instrument.

---

# 38. Performance Requirements

V0.1 should provide:

### Conductivity

Target sensor capability:

- 10 to 20,000 µS/cm.
- 1 µS/cm resolution.
- ±1% FS sensor accuracy.

### TOF

The system should provide microsecond-level timing for the prototype HC-SR04 measurement.

The actual distance accuracy must be validated experimentally.

### User Interface

The system should respond to physical input without noticeable delay.

### Logging

A completed valid measurement should be written to the SD card without disrupting normal application operation.

---

# 39. Portability Requirements

The analyzer should:

- Operate from its internal rechargeable battery.
- Include an integrated display.
- Include physical controls.
- Use removable sensors.
- Operate without external computer connection.
- Operate without internet connectivity.

---

# 40. Maintainability Requirements

The system should allow:

- Sensor replacement.
- Firmware updates through USB.
- SD card replacement.
- Battery replacement where practical.
- Configuration changes.
- Calibration updates.
- Troubleshooting through system diagnostics.

The hardware and software should avoid unnecessary proprietary dependencies.

---

# 41. Prototype Development Requirements

The project shall follow a prototype-first approach.

### Phase 1

Prove:

- EC communication.
- Ultrasonic timing.
- Distance calculation.
- Basic display.
- Measurement sequence.

### Phase 2

Add:

- RTC.
- SD logging.
- Physical controls.
- Battery operation.

### Phase 3

Add:

- Wi-Fi.
- Web UI.
- REST API.
- Configuration.
- Calibration.

### Phase 4

Perform:

- Repeatability testing.
- Long-duration testing.
- Battery testing.
- Mechanical integration.
- End-to-end validation.

---

# 42. Production Upgrade Considerations

V0.1 shall not be treated as the final production design.

Potential future improvements include:

- Laboratory-grade ultrasonic transducer.
- Dedicated ultrasonic receiver.
- Precision acoustic signal processing.
- Improved TOF timing hardware.
- Sample cell optimization.
- Temperature compensation.
- Improved conductivity measurement architecture.
- Custom PCB.
- Improved power management.
- Improved enclosure.
- Better sensor connectors.
- Calibration automation.
- Cloud connectivity if required.
- OTA firmware updates.

---

# 43. Explicit V0.1 Exclusions

The following are outside the V0.1 product scope:

- Production-grade liquid ultrasonic measurement.
- Advanced acoustic signal processing.
- Dedicated ultrasonic analog front end.
- Battery current measurement.
- Battery power measurement.
- INA226.
- Dedicated fuel gauge.
- Detailed battery analytics.
- Cloud platform.
- Internet connectivity.
- Mobile application.
- Web OTA.
- GPS.
- Cellular communication.
- Touchscreen.
- Automated sample identification.
- Industrial certification.
- Production EMC certification.
- Final production enclosure certification.

---

# 44. Product Acceptance Criteria

The EC-TOF Analyzer V0.1 is considered functionally complete when:

## Measurement

- Conductivity can be measured.
- TOF can be measured.
- Distance can be calculated.
- Measurements can be validated.
- Measurement failures are detected.

## User Interface

- TFT displays measurements.
- Rotary encoder works.
- START button works.
- BACK button works.
- Battery percentage is displayed.
- System warnings are displayed.

## Data

- RTC provides timestamps.
- Measurements can be saved to SD.
- CSV format is correct.
- SD failures are handled safely.

## Connectivity

- Wi-Fi AP starts.
- Web dashboard loads.
- REST API works.
- Current measurement can be retrieved.
- Measurement can be started through the API.

## Power

- Battery powers the system.
- USB-C charging works.
- Battery voltage is measured.
- Battery percentage is estimated.
- Low-battery state is detected.
- Critical-battery state is detected.

## Reliability

- Sensor communication failures are detected.
- Measurement timeouts are handled.
- The system returns to a known state after errors.
- Extended operation does not cause uncontrolled crashes.

---

# 45. Definition of Done

V0.1 is complete when a user can:

```text
Power ON
    │
    ▼
Prepare Sample
    │
    ▼
Position Sensors
    │
    ▼
Press START
    │
    ▼
Measure TOF
    │
    ▼
Calculate Distance
    │
    ▼
Measure Conductivity
    │
    ▼
Validate Result
    │
    ▼
View Result
    │
    ▼
Store Result
    │
    ▼
View Result Through Wi-Fi
```

The complete workflow must operate reliably from the internal battery.

---

# 46. Final Product Definition

The **EC-TOF Analyzer V0.1** is a portable, battery-powered prototype that combines electrical conductivity measurement with an ultrasonic TOF measurement workflow.

Its primary purpose is to validate the complete measurement concept and establish a modular platform for future development.

The V0.1 architecture intentionally separates the conductivity and ultrasonic subsystems from the application layer.

This allows the prototype to use affordable off-the-shelf hardware while preserving a clear upgrade path toward a production-grade ultrasonic measurement system.

The final V0.1 product consists of:

```text
┌─────────────────────────────────────────────┐
│              EC-TOF ANALYZER                │
│                                             │
│  EC Measurement                             │
│       │                                     │
│       ▼                                     │
│  SEN0707 + RS485                            │
│                                             │
│  Ultrasonic TOF                             │
│       │                                     │
│       ▼                                     │
│  HC-SR04 Prototype                          │
│                                             │
│  ESP32-S3                                   │
│       │                                     │
│       ├── TFT Display                       │
│       ├── Physical Controls                 │
│       ├── RTC                               │
│       ├── MicroSD                           │
│       ├── Wi-Fi                             │
│       └── Battery Indicator                 │
│                                             │
│  Rechargeable Battery                       │
│                                             │
└─────────────────────────────────────────────┘
```

This document defines the product baseline for **EC-TOF Analyzer V0.1**.