# EC-TOF Analyzer

## System Architecture

Version: 0.1
Status: Prototype Planning
Related Documents:

- `01-product-requirements.md`
- `02-technical-requirements.md`

---

# 1. Purpose

This document defines the system architecture of the EC-TOF Analyzer V0.1 prototype.

It describes:

- Overall system architecture
- Hardware architecture
- Software architecture
- Sensor interfaces
- Measurement data flow
- Measurement sequence
- Power architecture
- Communication architecture
- User interface architecture
- Data storage architecture
- Error handling
- Future expansion strategy

The architecture is designed to support rapid prototype development while allowing individual hardware components to be replaced in future versions.

---

# 2. Architecture Principles

The EC-TOF Analyzer shall follow these principles:

1. Use off-the-shelf hardware for V0.1.
2. Keep external sensors removable.
3. Separate hardware drivers from application logic.
4. Avoid unnecessary dependencies.
5. Keep sensor-specific code isolated.
6. Use local operation without internet dependency.
7. Protect the ESP32-S3 from incompatible signal levels.
8. Treat the HC-SR04 as a replaceable prototype sensor.
9. Store measurements using an open format.
10. Design the software so production hardware can replace prototype hardware later.

---

# 3. High-Level System Architecture

The complete system can be represented as:

```text
                         EC-TOF ANALYZER
                              │
                              │
                       ┌──────▼──────┐
                       │  ESP32-S3   │
                       │ Main MCU    │
                       └──────┬──────┘
                              │
          ┌───────────────────┼────────────────────┐
          │                   │                    │
          ▼                   ▼                    ▼
      MEASUREMENT           USER UI             STORAGE
       SYSTEM                SYSTEM              SYSTEM
          │                   │                    │
     ┌────┴────┐         ┌────┴────┐          ┌────┴────┐
     │         │         │         │          │         │
     ▼         ▼         ▼         ▼          ▼         ▼
    EC        TOF       TFT      Controls     SD       RTC
 Sensor     Sensor    Display    Encoder     Card     DS3231
     │         │
     │         │
    RS485     GPIO
     │         │
     ▼         ▼
 SEN0707    HC-SR04


                      ESP32-S3
                         │
                         ▼
                       Wi-Fi
                         │
                         ▼
                    Local Web UI
```

---

# 4. Main Subsystems

The system consists of the following major subsystems:

```text
1. Main Controller
2. Conductivity Measurement
3. Ultrasonic Measurement
4. Display
5. User Input
6. Real-Time Clock
7. Data Storage
8. Wi-Fi and Web Interface
9. Power Management
10. Battery
11. Configuration
12. Calibration
```

---

# 5. Hardware Architecture

## 5.1 Main Controller

The ESP32-S3 is the central controller.

Responsibilities:

- Execute firmware
- Manage sensors
- Process measurements
- Control display
- Process user input
- Manage storage
- Maintain system state
- Host Wi-Fi access point
- Host web server
- Manage configuration
- Manage calibration
- Report system errors

The ESP32-S3 shall not directly contain application logic for individual sensors.

Sensor-specific operations shall be implemented through drivers.

---

# 6. Conductivity Architecture

The conductivity subsystem consists of:

```text
┌──────────────┐
│   SEN0707    │
│ EC Sensor    │
└──────┬───────┘
       │
       │ RS485
       │
┌──────▼───────┐
│   MAX3485    │
│ RS485        │
│ Transceiver  │
└──────┬───────┘
       │
       │ UART
       │
┌──────▼───────┐
│   ESP32-S3   │
└──────────────┘
```

The SEN0707 remains physically removable.

The MAX3485 provides the electrical interface between the ESP32-S3 UART and the RS485 bus.

The EC application does not directly manipulate UART or RS485 signals.

The software layers are:

```text
EC Application
      ↓
EC Driver
      ↓
Modbus RTU Layer
      ↓
RS485 Driver
      ↓
UART
      ↓
MAX3485
      ↓
SEN0707
```

---

# 7. Ultrasonic Architecture

The V0.1 ultrasonic subsystem consists of:

```text
┌──────────────┐
│   HC-SR04    │
│ Ultrasonic   │
│ Sensor       │
└──────┬───────┘
       │
       ├── TRIG
       │
       └── ECHO
              │
              ▼
       Level Shifting
              │
              ▼
         ESP32-S3
```

The ultrasonic driver shall hide the HC-SR04-specific implementation.

The application shall interact with the ultrasonic subsystem through an abstract interface.

Example:

```text
Application
    │
    ▼
TOF Measurement Service
    │
    ▼
Ultrasonic Driver
    │
    ▼
HC-SR04
```

This allows the HC-SR04 to be replaced later.

---

# 8. Ultrasonic Future Architecture

The production system may use a different ultrasonic arrangement.

Possible future architecture:

```text
ESP32-S3
    │
    ▼
TOF Controller
    │
    ├───────────────┐
    ▼               ▼
TX Driver       RX Amplifier
    │               │
    ▼               ▼
TX Transducer   RX Transducer
```

The application layer should not need to change when moving from the HC-SR04 to a dedicated ultrasonic transducer system.

Only the lower-level ultrasonic hardware and driver should change.

---

# 9. Display Architecture

The TFT display is connected to the ESP32-S3 through SPI.

```text
ESP32-S3
    │
    │ SPI
    ▼
ST7796 TFT
    │
    ▼
480 × 320 Display
```

The display subsystem shall provide an abstraction between the application and the physical display.

Software architecture:

```text
Application State
      ↓
UI Manager
      ↓
Display Renderer
      ↓
ST7796 Driver
      ↓
SPI
      ↓
TFT
```

The application should not directly draw pixels from sensor drivers.

---

# 10. User Input Architecture

The input system consists of:

- Rotary encoder
- Encoder push button
- START button
- BACK button

Architecture:

```text
Physical Controls
       │
       ▼
GPIO
       │
       ▼
Input Driver
       │
       ▼
Input Manager
       │
       ▼
Application Events
```

Example events:

```text
ENCODER_UP
ENCODER_DOWN
ENCODER_SELECT
START_PRESSED
BACK_PRESSED
```

This event-based design prevents individual GPIO implementations from being spread throughout the application.

---

# 11. RTC Architecture

The DS3231 provides the system clock.

```text
ESP32-S3
    │
    │ I2C
    ▼
DS3231
    │
    ▼
Date / Time
```

The RTC service shall provide:

```text
getDateTime()
setDateTime()
isAvailable()
```

The rest of the application should not directly access the I2C bus.

---

# 12. Storage Architecture

The MicroSD card provides persistent measurement storage.

```text
Measurement Service
        │
        ▼
   Storage Manager
        │
        ▼
    CSV Manager
        │
        ▼
      SPI SD
        │
        ▼
     MicroSD
```

The storage manager shall handle:

- File creation
- Directory creation
- CSV headers
- Record writing
- File flushing
- Storage errors
- File retrieval
- Data export

---

# 13. Data Flow

The primary measurement data flow is:

```text
             ┌──────────────┐
             │ EC Sensor    │
             └──────┬───────┘
                    │
                    ▼
             EC Driver
                    │
                    ▼
             EC Measurement
                    │
                    │
                    ▼
              Measurement
                Manager
                    ▲
                    │
                    │
             TOF Measurement
                    ▲
                    │
                TOF Driver
                    ▲
                    │
             Ultrasonic Sensor
```

The measurement manager combines the sensor results into one measurement record.

Example internal data structure:

```cpp
struct Measurement
{
    uint32_t id;
    DateTime timestamp;

    float conductivity_uS_cm;
    float tof_us;
    float distance_mm;

    MeasurementStatus status;
};
```

---

# 14. Measurement Sequence

The standard measurement sequence shall be:

```text
                 IDLE
                   │
                   │ START
                   ▼
                STARTING
                   │
                   ▼
          Trigger Ultrasonic
                   │
                   ▼
            Wait for Echo
              │         │
              │         │
           Valid      Timeout
              │         │
              ▼         ▼
          Calculate    ERROR
             TOF          │
              │           ▼
              ▼         READY
        Calculate
          Distance
              │
              ▼
        Read Conductivity
              │
              ▼
        Validate Results
              │
              ├───────────────┐
              │               │
            Valid           Invalid
              │               │
              ▼               ▼
        Create Record        ERROR
              │
              ▼
        Update Display
              │
              ▼
          Write to SD
              │
              ▼
             READY
```

---

# 15. Measurement State Machine

The firmware shall implement the measurement workflow as a state machine.

Recommended states:

```text
SYSTEM_INIT
IDLE
MEASUREMENT_START
TOF_TRIGGER
TOF_WAIT
TOF_PROCESS
EC_READ
VALIDATE
DISPLAY_RESULT
LOG_RESULT
MEASUREMENT_COMPLETE
MEASUREMENT_ERROR
```

State transitions shall be deterministic.

A failed operation shall not leave the system permanently stuck in a measurement state.

---

# 16. Measurement Record Lifecycle

A measurement follows this lifecycle:

```text
Sensor Data
    ↓
Raw Measurement
    ↓
Validation
    ↓
Processed Measurement
    ↓
Measurement Object
    ↓
Display
    ↓
Storage
    ↓
Web API
```

The same validated measurement object should be used by:

- TFT UI
- Web UI
- SD logger
- Measurement history

This prevents the different interfaces from calculating different values.

---

# 17. Data Ownership

Each subsystem shall have clear ownership.

| Data                 | Owner                            |
| -------------------- | -------------------------------- |
| Conductivity reading | EC Driver / Measurement Manager  |
| TOF                  | TOF Driver / Measurement Manager |
| Distance             | Measurement Manager              |
| Timestamp            | RTC Service                      |
| Measurement record   | Measurement Manager              |
| Stored measurement   | Storage Manager                  |
| Calibration values   | Calibration Manager              |
| Device configuration | Configuration Manager            |
| UI state             | UI Manager                       |
| Battery state        | Power Manager                    |
| System state         | System Manager                   |

---

# 18. Software Architecture

The recommended software architecture is:

```text
┌─────────────────────────────────────────┐
│              User Interfaces            │
│                                         │
│       TFT UI              Web UI        │
└────────────────┬───────────────┬────────┘
                 │               │
                 ▼               ▼
┌─────────────────────────────────────────┐
│           Application Services          │
│                                         │
│ Measurement │ Calibration │ Config      │
│ System      │ Power       │ History     │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│              Device Services            │
│                                         │
│ EC │ TOF │ RTC │ Storage │ Input │ WiFi │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│              Hardware Drivers           │
│                                         │
│ UART │ RS485 │ GPIO │ SPI │ I2C │ RMT   │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│                 Hardware                │
└─────────────────────────────────────────┘
```

---

# 19. Software Layer Responsibilities

## 19.1 Hardware Drivers

Responsible for direct hardware communication.

Examples:

```text
GPIO
SPI
I2C
UART
RS485
RMT
SD
TFT
```

Drivers should not contain application decisions.

---

## 19.2 Device Services

Responsible for converting hardware interfaces into usable device functions.

Examples:

```text
EC Service
TOF Service
RTC Service
Storage Service
Input Service
Power Service
```

---

## 19.3 Application Services

Responsible for system behavior.

Examples:

```text
Measurement Manager
Calibration Manager
Configuration Manager
System Manager
History Manager
```

---

## 19.4 User Interfaces

Responsible only for presentation and user interaction.

Interfaces:

```text
TFT UI
Web UI
```

Both interfaces should consume the same application state.

---

# 20. FreeRTOS Architecture

The system may use the following tasks:

```text
┌──────────────────────────────────┐
│          FreeRTOS System         │
│                                  │
│  System Task                     │
│  Measurement Task                │
│  UI Task                         │
│  Storage Task                    │
│  Web Task                        │
│  Input Task                      │
└──────────────────────────────────┘
```

The exact number of tasks may be reduced during implementation if simpler synchronization is sufficient.

---

# 21. Task Responsibilities

### System Task

Responsible for:

- Startup
- System state
- Error monitoring
- Power state

### Measurement Task

Responsible for:

- Measurement sequence
- Sensor coordination
- Validation
- Measurement result

### UI Task

Responsible for:

- Display updates
- Menu rendering
- UI state

### Storage Task

Responsible for:

- SD operations
- CSV writes
- Data export

### Web Task

Responsible for:

- HTTP server
- API requests
- Web interface

### Input Task

Responsible for:

- Buttons
- Rotary encoder
- Input events

---

# 22. Inter-Task Communication

Tasks should communicate using FreeRTOS synchronization mechanisms.

Recommended:

```text
Input Task
    │
    ▼
Event Queue
    │
    ▼
Application

Measurement Task
    │
    ▼
Measurement Queue
    │
    ├──► UI Task
    │
    ├──► Storage Task
    │
    └──► Web API
```

Shared data shall be protected against concurrent access.

---

# 23. Power Architecture

The power system is divided into several voltage domains.

```text
                  Battery
                 3.7 V
                    │
                    ▼
              Power Switch
                    │
          ┌─────────┴─────────┐
          │                   │
          ▼                   ▼
      12 V Boost          5 V Rail
          │                   │
          ▼             ┌─────┴─────┐
       SEN0707          │           │
                        ▼           ▼
                       TFT       HC-SR04
                        
                    3.3 V Logic
                         │
                         ▼
                     ESP32-S3
```

The final regulator arrangement will be determined during hardware design.

---

# 24. Power Domain Separation

The following should be considered separate power domains:

```text
High Voltage Sensor Domain
    └── SEN0707

5 V Peripheral Domain
    ├── TFT
    └── HC-SR04

3.3 V Logic Domain
    ├── ESP32-S3
    ├── MAX3485
    ├── DS3231
    └── Control Inputs
```

Power domains should share a controlled common ground unless isolation is required by the final hardware design.

---

# 25. RS485 Architecture

The RS485 interface shall use a dedicated UART.

```text
ESP32-S3 UART
     │
     ▼
MAX3485
     │
     ├── A
     └── B
         │
         ▼
      SEN0707
```

The RS485 communication layer shall handle:

- TX
- RX
- Driver enable
- Receive enable where required
- Modbus timing
- CRC
- Timeout
- Retry

---

# 26. SPI Architecture

SPI peripherals include:

```text
ESP32-S3
    │
    ├── SPI → TFT
    │
    └── SPI → MicroSD
```

Each peripheral shall have an independent chip-select signal.

Where practical, the display and SD card should use separate SPI buses to simplify development and reduce bus contention.

The final bus allocation will be defined in `04-hardware-design.md`.

---

# 27. I2C Architecture

I2C will primarily be used for the DS3231 RTC.

The system intentionally avoids using I2C for external sensors.

Architecture:

```text
ESP32-S3
    │
    │ I2C
    ▼
DS3231
```

Future internal I2C devices may be added if electrical noise and bus requirements permit.

---

# 28. GPIO Architecture

GPIOs will be allocated according to functional groups.

Major GPIO groups:

```text
EC / RS485
├── UART TX
├── UART RX
└── RS485 Direction

Ultrasonic
├── TRIG
└── ECHO

User Input
├── Encoder A
├── Encoder B
├── Encoder Button
├── START
└── BACK

Display
├── SPI
├── CS
├── DC
├── RESET
└── Backlight

SD
├── SPI
└── CS
```

Final GPIO numbers shall be defined in the hardware design document.

---

# 29. Calibration Architecture

Calibration shall be isolated from measurement logic.

```text
User
 │
 ▼
Calibration UI
 │
 ▼
Calibration Manager
 │
 ├── EC Calibration
 │
 └── TOF Calibration
 │
 ▼
Configuration Storage
```

The measurement manager shall retrieve validated calibration parameters when performing calculations.

Calibration data shall not be hardcoded into the sensor drivers.

---

# 30. Configuration Architecture

Configuration follows:

```text
User
 │
 ├── TFT UI
 │
 └── Web UI
       │
       ▼
Configuration Manager
       │
       ▼
Non-Volatile Storage
```

The configuration manager is the single source of truth for system configuration.

---

# 31. Web Architecture

The ESP32-S3 will host the local web application.

```text
Phone / Laptop
      │
      │ Wi-Fi
      ▼
ESP32-S3 Access Point
      │
      ▼
HTTP Server
      │
      ├── Web UI
      │
      └── REST API
              │
              ▼
        Application Services
```

The web interface must not directly access sensor hardware.

---

# 32. Web Data Flow

For current measurements:

```text
Sensor
  ↓
Measurement Manager
  ↓
Current Measurement
  ↓
Web API
  ↓
HTTP Response
  ↓
Browser
```

For stored measurements:

```text
MicroSD
  ↓
Storage Manager
  ↓
History / Export Service
  ↓
Web API
  ↓
Browser
```

---

# 33. Measurement Data Flow

The complete measurement flow is:

```text
                  START
                    │
                    ▼
             Measurement Task
                    │
                    ▼
             Ultrasonic Driver
                    │
                    ▼
              TOF Measurement
                    │
                    ▼
             Distance Calculation
                    │
                    ▼
                EC Driver
                    │
                    ▼
            Conductivity Reading
                    │
                    ▼
             Result Validation
                    │
                    ▼
            Measurement Object
                    │
          ┌─────────┼─────────┐
          │         │         │
          ▼         ▼         ▼
        TFT       SD Card    Web API
          │         │         │
          └─────────┴─────────┘
                    │
                    ▼
                  READY
```

---

# 34. Error Architecture

Errors should propagate through defined layers.

Example:

```text
SEN0707
   │
   ▼
Modbus Error
   │
   ▼
EC Driver
   │
   ▼
Measurement Manager
   │
   ▼
System Error State
   │
   ├──► TFT
   ├──► Web UI
   └──► Log
```

A low-level hardware error must not directly manipulate the user interface.

---

# 35. Error Recovery

Recoverable errors should use controlled recovery.

Example:

```text
RS485 Timeout
      │
      ▼
Retry
      │
   ┌──┴──┐
   │     │
Success Failure
   │     │
   ▼     ▼
Continue  Error State
```

The number of retries shall be configurable.

Persistent failures shall be reported to the user.

---

# 36. Startup Sequence

The startup sequence shall be:

```text
Power ON
   │
   ▼
ESP32 Boot
   │
   ▼
Hardware Initialization
   │
   ├── GPIO
   ├── SPI
   ├── UART
   ├── I2C
   └── Timers
   │
   ▼
Peripheral Initialization
   │
   ├── TFT
   ├── RTC
   ├── SD
   ├── RS485
   └── Ultrasonic
   │
   ▼
Load Configuration
   │
   ▼
Initialize Services
   │
   ▼
Start Wi-Fi
   │
   ▼
Start Web Server
   │
   ▼
System Ready
```

A failure in an optional subsystem should not necessarily prevent the system from booting.

For example:

```text
SD unavailable
      ↓
System continues
      ↓
SD ERROR displayed
```

---

# 37. Shutdown and Power Loss

The system should attempt to maintain data integrity during power loss.

Before shutdown where possible:

- Flush SD buffers
- Save configuration changes
- Stop active measurements
- Disable unnecessary peripherals

Because sudden battery removal cannot always be predicted, measurement records should be written promptly after completion.

---

# 38. Security Architecture

V0.1 is a local laboratory prototype.

Security requirements are intentionally limited.

The system should:

- Avoid exposing the device to the public internet.
- Use a configurable Wi-Fi password.
- Avoid storing unnecessary personal information.
- Validate web API inputs.
- Reject malformed requests.
- Restrict configuration values to valid ranges.

Advanced authentication is outside V0.1 scope.

---

# 39. Enclosure Architecture

The enclosure shall physically separate:

```text
Front
├── TFT Display
├── Rotary Encoder
├── START Button
└── BACK Button

Side / Rear
├── Power Switch
├── USB-C
├── EC Sensor Connector
└── Ultrasonic Connector

Internal
├── ESP32-S3
├── Battery
├── Power Converters
├── RS485 Interface
├── SD Module
└── RTC
```

The exact enclosure layout will be defined during hardware development.

---

# 40. Sensor Replacement Architecture

A major architectural requirement is sensor replaceability.

The application should use interfaces rather than sensor-specific implementations.

Example:

```text
              Measurement Service
                     │
            ┌────────┴────────┐
            │                 │
        EC Interface      TOF Interface
            │                 │
            ▼                 ▼
       SEN0707 Driver    HC-SR04 Driver
                              │
                              │ Future
                              ▼
                     Industrial TOF Driver
```

The measurement service should remain unchanged when the underlying sensor changes.

---

# 41. V0.1 to Production Architecture

The expected evolution is:

```text
V0.1 Prototype
      │
      ├── ESP32 DevKit
      ├── HC-SR04
      ├── Module-based power
      ├── Module-based SD
      └── Development enclosure
      │
      ▼
Validated Prototype
      │
      ├── Confirm measurement method
      ├── Confirm sensor requirements
      ├── Measure accuracy
      └── Measure power consumption
      │
      ▼
Production Design
      │
      ├── Custom PCB
      ├── Final ultrasonic hardware
      ├── Optimized power system
      ├── Industrial connectors
      ├── Production enclosure
      └── Validation / certification
```

The production design shall not be started until the V0.1 prototype has demonstrated the measurement concept.

---

# 42. Architecture Summary

The EC-TOF Analyzer uses the ESP32-S3 as the central processing platform.

The architecture is divided into independent subsystems:

```text
                         ┌──────────────┐
                         │   SEN0707    │
                         │ Conductivity │
                         └──────┬───────┘
                                │ RS485
                                ▼
                         ┌──────────────┐
                         │   ESP32-S3   │
                         │              │
        ┌────────────────┤ Main Control ├─────────────────┐
        │                │              │                 │
        │                └──────────────┘                 │
        │                       │                         │
        ▼                       ▼                         ▼
   HC-SR04                   TFT UI                    Wi-Fi
      │                       │                         │
      │                       │                         ▼
      │                       │                      Web UI
      │                       │
      ▼                       ▼
   TOF Data              User Controls

                         ESP32-S3
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
            SD Card                      DS3231
              │                           │
              ▼                           ▼
         Measurement                  Timestamp
           History
```

The key architectural decision is that the application operates on measurement abstractions rather than directly on sensor hardware.

This allows the prototype hardware to evolve without requiring a complete firmware rewrite.

---

# 43. Architecture Baseline

The following decisions are considered the V0.1 architecture baseline:

| Area                  | Decision                      |
| --------------------- | ----------------------------- |
| MCU                   | ESP32-S3                      |
| Firmware              | ESP-IDF                       |
| EC Sensor             | DFRobot SEN0707               |
| EC Interface          | RS485 Modbus RTU              |
| RS485 Transceiver     | MAX3485                       |
| Ultrasonic            | HC-SR04 prototype             |
| Ultrasonic Interface  | GPIO                          |
| Display               | 4-inch 480×320 ST7796 SPI TFT |
| Storage               | MicroSD SPI                   |
| RTC                   | DS3231 I2C                    |
| User Input            | Rotary encoder + START + BACK |
| Wi-Fi                 | ESP32-S3 Access Point         |
| Web                   | Local HTTP server             |
| Data Format           | CSV                           |
| Battery               | 3.7 V rechargeable battery    |
| EC Power              | Boosted supply                |
| Architecture          | Modular                       |
| External Sensors      | Removable                     |
| Production Ultrasonic | Not yet selected              |

This document defines the system-level architecture. Detailed GPIO assignments, wiring, connector selection, power converters, protection circuits, and the complete BOM will be defined in `04-hardware-design.md`.
