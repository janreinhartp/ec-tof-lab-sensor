# EC-TOF Analyzer V0.1
## System Architecture

**Version:** 0.1  
**Status:** Prototype  
**Target Platform:** ESP32-S3  
**Firmware:** ESP-IDF  
**Last Updated:** October 2026

---

# 1. Purpose

This document defines the overall system architecture for the EC-TOF Analyzer V0.1 prototype.

The architecture separates:

- Hardware
- Device drivers
- Sensor interfaces
- Application logic
- User interfaces
- Data storage
- Configuration
- System services

The architecture is designed to support rapid prototyping while keeping the system modular enough for future sensor and hardware upgrades.

---

# 2. System Overview

The EC-TOF Analyzer combines two measurement systems:

1. Electrical conductivity measurement.
2. Ultrasonic time-of-flight measurement.

The ESP32-S3 coordinates both systems and provides:

- Local display
- Physical controls
- Data logging
- Real-time clock
- Local Wi-Fi interface
- Battery level indication
- Configuration
- Calibration

High-level architecture:

```text
                         ┌─────────────────────────┐
                         │       USER INTERFACE    │
                         │                         │
                         │  TFT Display            │
                         │  Rotary Encoder         │
                         │  START / BACK Buttons   │
                         │  Local Web Interface    │
                         └────────────┬────────────┘
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │   APPLICATION LAYER     │
                         │                         │
                         │ Measurement Manager     │
                         │ Calibration Manager     │
                         │ Configuration Manager   │
                         │ System Manager          │
                         └────────────┬────────────┘
                                      │
             ┌────────────────────────┼────────────────────────┐
             │                        │                        │
             ▼                        ▼                        ▼
    ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
    │ SENSOR SERVICES │     │ SYSTEM SERVICES │     │ DATA SERVICES   │
    │                 │     │                 │     │                 │
    │ EC Manager      │     │ Power Manager   │     │ Storage Manager │
    │ TOF Manager     │     │ RTC Manager     │     │ Logging         │
    │                 │     │ Wi-Fi Manager   │     │ Configuration   │
    └────────┬────────┘     └────────┬────────┘     └────────┬────────┘
             │                       │                       │
             └───────────────────────┼───────────────────────┘
                                     │
                                     ▼
                         ┌─────────────────────────┐
                         │     DRIVER LAYER       │
                         │                         │
                         │ UART / RS485            │
                         │ SPI                     │
                         │ I2C                     │
                         │ GPIO                    │
                         │ ADC                     │
                         │ Timers                  │
                         └────────────┬────────────┘
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │       HARDWARE         │
                         └─────────────────────────┘
```

---

# 3. Hardware Architecture

## 3.1 Main Controller

The ESP32-S3-DevKitC-1-N8R8 is the primary controller.

It is responsible for:

- Sensor communication
- Measurement processing
- Display control
- User input
- Data logging
- RTC communication
- Wi-Fi
- Battery voltage measurement
- System state management

---

# 4. Hardware Block Diagram

```text
                           ┌──────────────────────┐
                           │   3.7 V Battery      │
                           │   ~5000 mAh          │
                           └──────────┬───────────┘
                                      │
                                      ▼
                           ┌──────────────────────┐
                           │ Power Distribution   │
                           └───────┬───────┬──────┘
                                   │       │
                    ┌──────────────┘       └───────────────┐
                    ▼                                      ▼
             ┌──────────────┐                       ┌──────────────┐
             │ 12 V Boost   │                       │ 5 V Supply  │
             └──────┬───────┘                       └──────┬───────┘
                    │                                      │
                    ▼                                      ├──► TFT
             ┌──────────────┐                              │
             │  SEN0707    │                              └──► HC-SR04
             │ Conductivity│
             └──────┬───────┘
                    │ RS485
                    ▼
             ┌──────────────┐
             │   MAX3485    │
             └──────┬───────┘
                    │ UART
                    ▼
             ┌─────────────────────────────────────────────┐
             │                 ESP32-S3                    │
             │                                             │
             │  UART ─────────► RS485 / EC                 │
             │  GPIO ─────────► HC-SR04                    │
             │  SPI ──────────► TFT                        │
             │  SPI ──────────► MicroSD                    │
             │  I2C ──────────► DS3231                     │
             │  ADC ──────────► Battery Voltage            │
             │  GPIO ─────────► Encoder / Buttons         │
             │  Wi-Fi ────────► Local Web Interface       │
             └─────────────────────────────────────────────┘
```

---

# 5. Interface Architecture

The system shall use different physical interfaces based on the requirements of each peripheral.

| Component | Interface | ESP32-S3 Connection |
|---|---|---|
| SEN0707 | RS485 / Modbus RTU | UART + MAX3485 |
| HC-SR04 | Trigger / Echo | GPIO |
| ST7796 TFT | SPI | SPI |
| MicroSD | SPI | SPI |
| DS3231 | I2C | I2C |
| Battery voltage | ADC | ADC GPIO |
| Rotary encoder | GPIO | GPIO |
| START button | GPIO | GPIO |
| BACK button | GPIO | GPIO |
| Wi-Fi | Integrated | ESP32-S3 |

External sensor communication does not use I2C.

---

# 6. Software Architecture

The firmware shall use a layered architecture.

```text
┌───────────────────────────────────────────┐
│              USER INTERFACE                │
│                                           │
│ TFT UI + Input + Web UI                   │
└────────────────────┬──────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────┐
│           APPLICATION LAYER              │
│                                           │
│ Measurement Manager                       │
│ Calibration Manager                       │
│ Configuration Manager                     │
│ System Manager                            │
└────────────────────┬──────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────┐
│             SERVICE LAYER                 │
│                                           │
│ EC Manager                                │
│ TOF Manager                               │
│ Storage Manager                           │
│ RTC Manager                               │
│ Power Manager                             │
│ Wi-Fi Manager                             │
└────────────────────┬──────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────┐
│              DRIVER LAYER                 │
│                                           │
│ UART / RS485                              │
│ SPI                                       │
│ I2C                                       │
│ GPIO                                      │
│ ADC                                       │
│ Timer                                     │
└────────────────────┬──────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────┐
│               HARDWARE                    │
└───────────────────────────────────────────┘
```

---

# 7. Layer Responsibilities

## 7.1 User Interface Layer

The UI layer is responsible for presenting system information and receiving user input.

Components:

- TFT display
- Rotary encoder
- START button
- BACK button
- Web interface

The UI shall not directly communicate with sensor hardware.

---

## 7.2 Application Layer

The application layer controls the overall device behavior.

Primary components:

### Measurement Manager

Responsible for:

- Starting measurements
- Coordinating TOF
- Reading EC
- Validating results
- Creating measurement records
- Updating the UI
- Triggering logging

### Calibration Manager

Responsible for:

- EC calibration
- TOF calibration
- Calibration validation
- Saving calibration parameters

### Configuration Manager

Responsible for:

- Loading configuration
- Saving configuration
- Configuration validation
- Configuration versioning

### System Manager

Responsible for:

- Device startup
- System state
- Error management
- Service coordination
- Shutdown handling

---

# 8. Sensor Service Layer

## 8.1 EC Manager

The EC Manager provides a hardware-independent interface to the conductivity sensor.

Architecture:

```text
Measurement Manager
        │
        ▼
   IEcSensor
        │
        ▼
 Sen0707Driver
        │
        ▼
 Modbus RTU
        │
        ▼
    MAX3485
        │
        ▼
    SEN0707
```

The application shall not directly manipulate Modbus frames.

---

# 9. EC Sensor Abstraction

The firmware should define an interface similar to:

```cpp
class IEcSensor
{
public:
    virtual bool begin() = 0;

    virtual bool read(
        float& conductivity
    ) = 0;

    virtual bool calibrate() = 0;

    virtual bool isConnected() = 0;

    virtual ~IEcSensor() = default;
};
```

The SEN0707 implementation shall provide the actual Modbus communication.

This allows a different EC sensor to be introduced later without rewriting the measurement manager.

---

# 10. TOF Manager

The TOF Manager shall provide a hardware-independent interface for ultrasonic measurement.

Architecture:

```text
Measurement Manager
        │
        ▼
   ITofSensor
        │
        ▼
 HC-SR04 Driver
        │
        ▼
 Trigger / Echo
        │
        ▼
    HC-SR04
```

The HC-SR04 implementation is considered a prototype implementation only.

---

# 11. TOF Sensor Abstraction

The firmware should define an interface similar to:

```cpp
class ITofSensor
{
public:
    virtual bool begin() = 0;

    virtual bool measureTof(
        float& tofUs
    ) = 0;

    virtual bool calculateDistance(
        float tofUs,
        float& distanceMm
    ) = 0;

    virtual ~ITofSensor() = default;
};
```

The interface shall prevent the rest of the application from depending directly on HC-SR04-specific behavior.

---

# 12. TOF Measurement Model

For the V0.1 air-based prototype:

```text
Distance = TOF × Speed of Sound / 2
```

The actual implementation shall account for the units used by the timer.

For example:

```text
TOF = 12.482 µs

Speed of Sound ≈ 343 m/s

Distance ≈ 2.14 mm
```

The calculation shall be implemented in a dedicated TOF processing component.

The future liquid/acoustic measurement system shall use a separate acoustic model if required.

---

# 13. Measurement Manager

The Measurement Manager is the central component for measurement execution.

Measurement sequence:

```text
IDLE
  │
  ▼
START
  │
  ▼
TOF_TRIGGER
  │
  ▼
TOF_WAIT
  │
  ▼
TOF_PROCESS
  │
  ▼
EC_READ
  │
  ▼
VALIDATE
  │
  ▼
DISPLAY
  │
  ▼
LOG
  │
  ▼
READY
```

---

# 14. Measurement State Machine

The measurement manager shall maintain an explicit state.

Example:

```cpp
enum class MeasurementState
{
    IDLE,
    START,
    TOF_TRIGGER,
    TOF_WAIT,
    TOF_PROCESS,
    EC_READ,
    VALIDATE,
    DISPLAY,
    LOG,
    READY,
    ERROR
};
```

This prevents measurement logic from becoming distributed across multiple UI and sensor components.

---

# 15. Measurement Data Model

The application shall use a common measurement structure.

Example:

```cpp
struct Measurement
{
    uint64_t timestamp;

    float conductivity;
    float tofUs;
    float distanceMm;

    bool conductivityValid;
    bool tofValid;

    bool valid;
};
```

The data model may be expanded later.

---

# 16. RTC Architecture

The RTC service shall provide a hardware-independent interface.

Architecture:

```text
Application
    │
    ▼
RTC Manager
    │
    ▼
DS3231 Driver
    │
    ▼
I2C
    │
    ▼
DS3231
```

The rest of the application shall not directly access the I2C peripheral.

The RTC Manager shall provide:

- Current date/time
- Timestamp generation
- RTC status

---

# 17. Storage Architecture

Storage shall be abstracted from the application.

Architecture:

```text
Measurement Manager
        │
        ▼
 Storage Manager
        │
        ▼
   SD Driver
        │
        ▼
       SPI
        │
        ▼
     MicroSD
```

The Storage Manager shall handle:

- File creation
- File opening
- Record writing
- File closing
- Storage errors
- Export operations

A storage failure shall not crash the measurement application.

---

# 18. Data Logging Flow

```text
Measurement Complete
        │
        ▼
Create Measurement Record
        │
        ▼
Add RTC Timestamp
        │
        ▼
Validate Record
        │
        ▼
Storage Manager
        │
        ▼
Append CSV
        │
        ▼
Confirm Write
```

If storage fails:

```text
Storage Failure
      │
      ├── Show Warning
      ├── Keep Measurement in Memory
      └── Continue Device Operation
```

The exact temporary buffering strategy may be expanded later.

---

# 19. Display Architecture

The display shall be separated into:

```text
Application Data
      │
      ▼
Display Manager
      │
      ▼
UI Screens
      │
      ▼
ST7796 Driver
      │
      ▼
SPI
      │
      ▼
TFT
```

The Display Manager shall not perform sensor communication.

---

# 20. Input Architecture

Physical inputs shall be handled by a dedicated Input Manager.

Architecture:

```text
Encoder / Buttons
        │
        ▼
   GPIO Driver
        │
        ▼
 Input Manager
        │
        ▼
Application
```

The Input Manager shall provide logical events such as:

```text
ENCODER_CW
ENCODER_CCW
ENCODER_PRESS
START_PRESS
BACK_PRESS
```

This keeps GPIO handling separate from application behavior.

---

# 21. Power Architecture

The system shall provide a simple battery monitoring service.

The Power Manager shall not perform detailed power analysis.

Architecture:

```text
Battery
   │
   ├── Power System
   │
   └── Voltage Divider
           │
           ▼
        ESP32 ADC
           │
           ▼
      Power Manager
           │
           ▼
    Battery Percentage
           │
           ├── TFT
           │
           └── Web UI
```

---

# 22. Battery Data Model

The Power Manager should expose:

```cpp
enum class BatteryState
{
    NORMAL,
    LOW
};

struct PowerStatus
{
    float batteryVoltage;
    uint8_t batteryPercent;
    BatteryState state;
};
```

The battery percentage is an estimate based on battery voltage.

It shall not be treated as laboratory-grade state-of-charge information.

---

# 23. Battery Monitoring Behavior

The Power Manager shall:

1. Read the ADC.
2. Convert the ADC value to battery voltage.
3. Estimate battery percentage.
4. Determine battery state.
5. Publish the result to the application.
6. Update the UI.

Example:

```text
Battery Voltage
      │
      ▼
ADC Conversion
      │
      ▼
Voltage Calculation
      │
      ▼
Percentage Lookup
      │
      ▼
Battery State
      │
      ▼
UI / Web
```

The system shall provide a low battery warning.

A precision battery fuel gauge is not required for V0.1.

---

# 24. Wi-Fi Architecture

The ESP32-S3 shall operate as a local Wi-Fi access point.

Architecture:

```text
ESP32-S3
    │
    ▼
Wi-Fi Manager
    │
    ▼
HTTP Server
    │
    ├── Web UI
    │
    └── REST API
```

Recommended default:

```text
SSID: EC-TOF-Analyzer
IP:   192.168.4.1
```

The Wi-Fi system shall operate independently of sensor communication.

---

# 25. Web Application Architecture

The web application shall consume application data.

It shall not access hardware drivers directly.

```text
Browser
   │
   ▼
HTTP Server
   │
   ▼
REST API
   │
   ▼
Application Services
   │
   ├── Measurement Manager
   ├── Configuration Manager
   ├── Calibration Manager
   ├── Storage Manager
   └── Power Manager
```

---

# 26. REST API Architecture

Required endpoints:

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

Example system status response:

```json
{
    "battery_voltage": 3.82,
    "battery_percent": 52,
    "battery_state": "NORMAL"
}
```

No battery current or power values are required.

---

# 27. Configuration Architecture

Configuration shall be managed through a dedicated service.

```text
Application
    │
    ▼
Configuration Manager
    │
    ▼
ESP-IDF NVS
```

Configuration shall include:

- Device settings
- Measurement settings
- Calibration values
- Display settings
- Logging settings
- Sensor configuration

Configuration shall be versioned.

---

# 28. Calibration Architecture

Calibration shall be independent from the sensor drivers.

```text
User
 │
 ▼
Calibration UI
 │
 ▼
Calibration Manager
 │
 ├───────────────┐
 ▼               ▼
EC Calibration   TOF Calibration
 │               │
 ▼               ▼
Sensor           Reference
 │               │
 └───────┬───────┘
         ▼
 Configuration Manager
         │
         ▼
        NVS
```

Calibration parameters shall survive a reboot.

---

# 29. Task Architecture

The firmware may use the following FreeRTOS tasks:

```text
┌─────────────────────────────┐
│ Measurement Task            │
│ Measurement state machine    │
└─────────────────────────────┘

┌─────────────────────────────┐
│ UI Task                     │
│ TFT updates                 │
│ User interaction             │
└─────────────────────────────┘

┌─────────────────────────────┐
│ Storage Task                │
│ SD logging                  │
└─────────────────────────────┘

┌─────────────────────────────┐
│ Web Task                    │
│ HTTP server                 │
└─────────────────────────────┘

┌─────────────────────────────┐
│ System Task                 │
│ RTC / battery / health      │
└─────────────────────────────┘
```

The final implementation should avoid creating unnecessary tasks.

Simple operations may run inside an existing task instead of receiving their own task.

---

# 30. Inter-Task Communication

Where required, FreeRTOS mechanisms shall be used.

Possible mechanisms:

- Queues
- Mutexes
- Event groups
- Notifications
- Shared state with controlled access

Example:

```text
Measurement Task
       │
       ▼
Measurement Queue
       │
       ├────────► UI
       │
       ├────────► Storage
       │
       └────────► Web
```

The measurement result should have a single application-level source of truth.

---

# 31. Error Architecture

Errors shall be handled at the appropriate layer.

Example:

```text
Hardware Error
      │
      ▼
Driver
      │
      ▼
Sensor Service
      │
      ▼
Application
      │
      ├── UI Warning
      ├── Web Status
      └── Log
```

The application shall not expose raw hardware errors directly to the user where a readable error message can be provided.

---

# 32. System Startup Sequence

The device shall initialize in a controlled sequence.

Recommended startup:

```text
Power On
   │
   ▼
ESP32 Boot
   │
   ▼
System Initialization
   │
   ├── GPIO
   ├── SPI
   ├── I2C
   ├── UART
   ├── ADC
   └── Timers
   │
   ▼
Peripheral Initialization
   │
   ├── TFT
   ├── SD
   ├── RTC
   ├── RS485
   ├── EC Sensor
   └── TOF Sensor
   │
   ▼
Configuration Load
   │
   ▼
Wi-Fi Start
   │
   ▼
Application Services Start
   │
   ▼
READY
```

A failure in one non-critical peripheral should not necessarily prevent the device from starting.

For example:

```text
SD Failure
   │
   ▼
Show SD Warning
   │
   ▼
Continue Device Operation
```

---

# 33. Measurement Data Flow

Complete measurement flow:

```text
User presses START
        │
        ▼
Measurement Manager
        │
        ▼
Trigger TOF
        │
        ▼
Capture Echo
        │
        ▼
Calculate TOF
        │
        ▼
Calculate Distance
        │
        ▼
Read EC
        │
        ▼
Validate Measurement
        │
        ▼
Create Measurement Record
        │
        ├──────────────► Display
        │
        ├──────────────► Web API
        │
        └──────────────► Storage
```

---

# 34. Measurement Validation

Before a measurement is marked valid, the application shall verify:

- TOF completed successfully.
- TOF is within configured limits.
- Distance calculation succeeded.
- EC communication succeeded.
- EC value is within configured limits.
- Timestamp is available.

If validation fails:

```text
Measurement
    │
    ▼
Validation Failed
    │
    ├── Status = INVALID
    ├── Show Error
    └── Optional Log
```

---

# 35. Project Structure

Recommended firmware structure:

```text
ec-tof-analyzer/
│
├── main/
│   │
│   ├── main.cpp
│   │
│   ├── app/
│   │   ├── measurement_manager/
│   │   ├── calibration_manager/
│   │   ├── configuration_manager/
│   │   └── system_manager/
│   │
│   ├── sensors/
│   │   ├── ec/
│   │   │   ├── iec_sensor.h
│   │   │   └── sen0707_driver.cpp
│   │   │
│   │   └── tof/
│   │       ├── itof_sensor.h
│   │       └── hc_sr04_driver.cpp
│   │
│   ├── drivers/
│   │   ├── rs485/
│   │   ├── uart/
│   │   ├── spi/
│   │   ├── i2c/
│   │   ├── gpio/
│   │   ├── adc/
│   │   └── timer/
│   │
│   ├── display/
│   ├── input/
│   ├── storage/
│   ├── rtc/
│   ├── web/
│   ├── power/
│   ├── system/
│   └── common/
│
└── components/
```

---

# 36. Dependency Rules

The architecture shall follow these dependency rules:

### UI

Can depend on:

- Application data
- UI services

Should not depend directly on:

- Sensor drivers
- UART
- SPI
- I2C

### Application

Can depend on:

- Service interfaces
- Data models

Should not depend directly on:

- GPIO registers
- SPI transactions
- UART transactions

### Services

Can depend on:

- Driver interfaces
- Hardware abstractions

### Drivers

Can depend on:

- ESP-IDF hardware APIs

Drivers should not depend on:

- UI
- Web
- Measurement Manager

---

# 37. Prototype-to-Production Strategy

The V0.1 architecture intentionally separates hardware-specific implementations from application logic.

For example:

```text
Current:

ITofSensor
    │
    ▼
HC-SR04 Driver

Future:

ITofSensor
    │
    ▼
Production Acoustic Driver
```

The Measurement Manager should not require significant changes when the TOF hardware changes.

The same principle applies to the conductivity sensor:

```text
IEcSensor
    │
    ▼
Sen0707Driver
```

Future:

```text
IEcSensor
    │
    ▼
ProductionEcDriver
```

---

# 38. Reliability Principles

The firmware shall follow these principles:

1. Sensor failures must not crash the application.
2. SD failures must not crash the measurement system.
3. Web interface failures must not stop measurements.
4. UI failures must not directly affect sensor drivers.
5. Hardware-specific code shall remain isolated.
6. Measurement data shall have a single application-level representation.
7. Configuration shall be persistent.
8. Calibration data shall be persistent.
9. External sensors shall remain removable.
10. Interfaces shall be replaceable where practical.

---

# 39. V0.1 Architecture Boundary

The V0.1 architecture intentionally focuses on:

```text
EC Measurement
       +
Ultrasonic TOF
       +
Distance Calculation
       +
Timestamp
       +
Data Logging
       +
Local Display
       +
Physical Controls
       +
Local Wi-Fi
       +
Simple Battery Indicator
```

The following are outside the V0.1 architecture:

- Cloud services
- Cellular communication
- Remote monitoring
- Precision battery fuel gauging
- Battery current measurement
- Battery power measurement
- Production-grade acoustic electronics
- Advanced analytics
- User authentication
- Multi-device management
- OTA firmware management

---

# 40. Architecture Success Criteria

The architecture shall be considered successful when:

- Each major hardware subsystem has a clear interface.
- Application logic is independent of specific sensor implementations.
- EC communication is isolated behind an EC interface.
- TOF communication is isolated behind a TOF interface.
- Hardware drivers are separated from application logic.
- Measurement data has a consistent structure.
- UI and web interfaces consume application data.
- Storage is isolated behind a storage service.
- RTC access is isolated behind an RTC service.
- Battery monitoring is isolated behind a power service.
- Sensor failures are recoverable.
- The prototype can be upgraded without rewriting the entire application.

---

# 41. Final Architecture

The final V0.1 architecture can be summarized as:

```text
                         ┌─────────────────────────┐
                         │        USER            │
                         └────────────┬────────────┘
                                      │
                  ┌───────────────────┴───────────────────┐
                  │                                       │
                  ▼                                       ▼
          ┌───────────────┐                       ┌───────────────┐
          │ TFT + Inputs  │                       │ Web Browser   │
          └───────┬───────┘                       └───────┬───────┘
                  │                                       │
                  └──────────────────┬────────────────────┘
                                     ▼
                         ┌─────────────────────────┐
                         │   APPLICATION LAYER     │
                         │                         │
                         │ Measurement Manager     │
                         │ Calibration Manager     │
                         │ Configuration Manager   │
                         │ System Manager          │
                         └────────────┬────────────┘
                                      │
            ┌─────────────────────────┼─────────────────────────┐
            │                         │                         │
            ▼                         ▼                         ▼
     ┌─────────────┐          ┌─────────────┐          ┌─────────────┐
     │ EC Manager  │          │ TOF Manager │          │   Services  │
     │             │          │             │          │             │
     │ SEN0707     │          │ HC-SR04     │          │ Storage     │
     │ Modbus RTU  │          │             │          │ RTC         │
     └──────┬──────┘          └──────┬──────┘          │ Power       │
            │                        │                 │ Wi-Fi       │
            ▼                        ▼                 └──────┬──────┘
     ┌─────────────┐          ┌─────────────┐                │
     │ MAX3485     │          │ GPIO/Timer  │                │
     └──────┬──────┘          └──────┬──────┘                │
            │                        │                        │
            ▼                        ▼                        ▼
       SEN0707                   HC-SR04                ESP32-S3
                                                        Peripherals
                                                           │
                         ┌─────────────────────────────────┼──────────────┐
                         │                                 │              │
                         ▼                                 ▼              ▼
                       TFT                              MicroSD         DS3231
                         │
                         ▼
                    User Output
```

The architecture keeps the prototype simple while preserving clear boundaries for future production hardware.