# EC-TOF Analyzer V0.1
## Implementation Plan

**Document Version:** 1.0  
**Project Version:** V0.1  
**Platform:** ESP32-S3  
**Framework:** ESP-IDF  
**Language:** C++  
**Status:** Implementation Baseline

---

## 1. Purpose

This document defines the implementation sequence for the EC-TOF Analyzer V0.1 prototype.

The goal is to build the system incrementally.

Each major hardware and software component must be tested before integration.

The implementation must prioritize:

- Working measurement functions
- Modular firmware
- Replaceable sensor drivers
- Reliable data logging
- Local web access
- Battery-powered operation
- Simple physical controls
- Clear error handling
- Easy future hardware replacement

V0.1 is a prototype.

The implementation must not introduce production-level complexity unless it is required for the prototype.

---

# 2. Development Strategy

Development follows a bottom-up approach.

```text
Hardware Bring-Up
       ↓
Basic Drivers
       ↓
Individual Sensor Drivers
       ↓
Measurement Functions
       ↓
Measurement Manager
       ↓
Display and Controls
       ↓
Data Logging
       ↓
Calibration
       ↓
Wi-Fi / Web Interface
       ↓
System Integration
       ↓
Validation
```

Each phase should produce a testable result.

Do not integrate multiple untested subsystems at the same time.

---

# 3. Implementation Phases

## Phase 0: Project Setup

### Objective

Create the initial ESP-IDF project structure.

### Tasks

- Create ESP-IDF project.
- Configure ESP32-S3 target.
- Configure C++ support.
- Create project directory structure.
- Configure Git repository.
- Add `.gitignore`.
- Create initial `README.md`.
- Configure build and flash commands.
- Verify serial logging.
- Verify firmware flashing.

### Initial structure

```text
ec-tof-analyzer/
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

### Exit Criteria

- Firmware builds successfully.
- ESP32-S3 flashes successfully.
- Serial console works.
- Basic application startup message appears.

---

# 4. Phase 1: ESP32-S3 Hardware Bring-Up

## Objective

Verify the main controller and basic GPIO functionality.

### Tasks

- Configure board.
- Verify 3.3 V logic.
- Test GPIO output.
- Test GPIO input.
- Test button input.
- Test system reset.
- Test serial communication.
- Verify available SPI pins.
- Verify available UART pins.
- Verify I2C pins.
- Verify interrupt-capable GPIOs.

### Basic test firmware

Implement:

```text
System Start
    ↓
GPIO Initialization
    ↓
Button Test
    ↓
Output Test
    ↓
Serial Logging
```

### Exit Criteria

- All required GPIOs are identified.
- No pin conflicts exist.
- Buttons can be detected.
- Outputs respond correctly.

---

# 5. Phase 2: Display Implementation

## Objective

Bring up the 4" 480x320 ST7796 SPI display.

### Tasks

- Implement SPI driver configuration.
- Initialize ST7796.
- Configure display resolution.
- Test screen initialization.
- Test text rendering.
- Test basic graphics.
- Test screen refresh.
- Implement display abstraction.
- Create initial UI layout.

### Initial screen

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

### Exit Criteria

- Display initializes reliably.
- Text is readable.
- Screen updates without corruption.
- Display driver is isolated from application logic.

---

# 6. Phase 3: Physical Input

## Objective

Implement the rotary encoder and physical buttons.

### Inputs

- Rotary encoder clockwise
- Rotary encoder counter-clockwise
- Rotary encoder push
- START button
- BACK button

### Tasks

- Configure GPIO inputs.
- Implement debouncing.
- Detect encoder rotation.
- Detect encoder button.
- Detect START.
- Detect BACK.
- Create `InputManager`.
- Generate high-level input events.

### Example events

```text
ENCODER_CW
ENCODER_CCW
ENCODER_PRESS
START_PRESS
BACK_PRESS
```

### Exit Criteria

- All controls respond correctly.
- No significant false triggering.
- UI can consume high-level input events.

---

# 7. Phase 4: RTC Implementation

## Objective

Implement the DS3231 RTC.

### Tasks

- Implement I2C driver.
- Initialize DS3231.
- Read current date/time.
- Set date/time.
- Validate RTC data.
- Implement `RtcManager`.
- Add timestamp formatting.

### Required timestamp format

```text
YYYY-MM-DDTHH:MM:SS
```

Example:

```text
2026-10-08T15:32:10
```

### Exit Criteria

- RTC can be read.
- RTC can be configured.
- Timestamp is stable.
- RTC errors are detected.

---

# 8. Phase 5: MicroSD Storage

## Objective

Implement reliable measurement storage.

### Tasks

- Configure SD card over SPI.
- Mount filesystem.
- Detect card insertion/startup availability.
- Create directory structure.
- Create CSV files.
- Append measurements.
- Read stored measurements.
- Handle SD errors.

### Directory structure

```text
/ECTOF/
├── config/
├── data/
│   └── YYYY/
│       └── MM/
│           └── DD.csv
└── logs/
```

### CSV format

```text
timestamp,conductivity,tof_us,distance_mm,status
```

Example:

```text
2026-10-08T15:32:10,4820,12.482,18.73,VALID
```

### Exit Criteria

- SD card mounts successfully.
- Files can be created.
- Measurements can be appended.
- Files can be read after reboot.
- SD failure does not crash the system.

---

# 9. Phase 6: UART and RS485

## Objective

Create the communication layer required by the SEN0707.

### Architecture

```text
ESP32-S3 UART
      |
      v
MAX3485
      |
      v
RS485
      |
      v
SEN0707
```

### Tasks

- Configure UART.
- Configure RS485 direction control.
- Implement transmit.
- Implement receive.
- Implement timeout.
- Implement CRC handling.
- Implement frame validation.
- Implement communication error detection.

### Required errors

```text
TIMEOUT
CRC_ERROR
INVALID_FRAME
UART_ERROR
SENSOR_DISCONNECTED
```

### Exit Criteria

- ESP32-S3 communicates with MAX3485.
- Modbus frames can be transmitted.
- Responses can be received.
- Invalid responses are detected.

---

# 10. Phase 7: SEN0707 EC Sensor

## Objective

Implement the conductivity sensor driver.

### Tasks

- Create `IEcSensor` interface.
- Create `Sen0707Driver`.
- Implement Modbus communication.
- Read conductivity.
- Validate returned data.
- Handle sensor timeout.
- Handle invalid readings.
- Convert units where required.

### Internal flow

```text
MeasurementManager
       ↓
IEcSensor
       ↓
Sen0707Driver
       ↓
Modbus
       ↓
RS485
       ↓
SEN0707
```

### Example API

```cpp
bool begin();

bool readConductivity(float& conductivity);

bool isConnected();

EcSensorStatus getStatus();
```

### Exit Criteria

- Conductivity can be read reliably.
- Sensor communication errors are handled.
- Sensor driver is independent from the UI.

---

# 11. Phase 8: EC Measurement Validation

## Objective

Verify conductivity measurements before integrating them into the full measurement cycle.

### Tests

- Low conductivity sample.
- Medium conductivity sample.
- High conductivity sample.
- Repeated measurements.
- Sensor disconnect.
- Communication timeout.
- Invalid response.
- Calibration solution.

### Required behavior

A failed sensor reading must not generate a `VALID` measurement.

### Exit Criteria

- Conductivity readings are stable enough for prototype use.
- Error states are correctly reported.
- Sensor data is available through the application layer.

---

# 12. Phase 9: HC-SR04 TOF Driver

## Objective

Implement the initial ultrasonic timing subsystem.

### Important Scope

The HC-SR04 is used only for V0.1 feasibility and prototype development.

It must not be treated as the final liquid or laboratory acoustic TOF sensor.

The TOF software must remain hardware-independent.

### Tasks

- Configure TRIG GPIO.
- Configure ECHO GPIO.
- Implement ECHO level shifting.
- Implement microsecond timing.
- Implement trigger pulse.
- Measure ECHO duration.
- Implement timeout.
- Reject invalid measurements.

### Measurement sequence

```text
TRIGGER
   ↓
WAIT FOR ECHO HIGH
   ↓
START TIMER
   ↓
WAIT FOR ECHO LOW
   ↓
STOP TIMER
   ↓
CALCULATE TOF
```

### Required timeout

The driver must never wait indefinitely for ECHO.

### Exit Criteria

- Trigger works.
- Echo timing works.
- Timeout works.
- Repeated measurements are possible.
- Driver returns a structured measurement result.

---

# 13. Phase 10: TOF Processing

## Objective

Convert the measured ultrasonic timing into distance for the prototype.

### V0.1 relationship

For air-based HC-SR04 testing:

```text
Distance = TOF × Speed of Sound / 2
```

The factor of two accounts for the outbound and return path.

### Important Limitation

The speed-of-sound assumption is environment-dependent.

The V0.1 implementation must not hard-code this calculation as the final method for the future liquid/acoustic TOF system.

The TOF processing layer must allow the future sensor implementation to provide its own conversion.

### Exit Criteria

- TOF values are measured.
- Distance calculation works for controlled air testing.
- Sensor timeout is handled.
- TOF abstraction remains replaceable.

---

# 14. Phase 11: Measurement Manager

## Objective

Integrate EC and TOF measurements into one controlled measurement cycle.

### State machine

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

### Error path

```text
Any measurement failure
        ↓
     ERROR
        ↓
   Display Error
        ↓
      READY
```

### Measurement result

```cpp
struct MeasurementResult
{
    uint64_t id;
    Timestamp timestamp;

    float conductivity;
    float tof_us;
    float distance_mm;

    MeasurementStatus status;
};
```

### Exit Criteria

- START initiates a measurement.
- TOF is measured.
- EC is measured.
- Results are validated.
- Results are displayed.
- Valid results are logged.

---

# 15. Phase 12: Calibration

## Objective

Implement calibration management.

## 15.1 EC Calibration

Use the manufacturer's recommended calibration procedure and supplied calibration solution.

### Tasks

- Create EC calibration screen.
- Read current sensor value.
- Allow calibration value entry.
- Calculate/store calibration parameters as required.
- Save calibration configuration to NVS.
- Display calibration status.

### Validation

Compare sensor reading against the known calibration solution.

---

## 15.2 TOF Calibration

Use a known physical distance.

### Procedure

```text
Known Distance
      ↓
Measure TOF
      ↓
Calculate Calibration Factor
      ↓
Store Calibration
      ↓
Validate
```

### Exit Criteria

- Calibration data survives reboot.
- Calibration can be changed.
- Invalid calibration values are rejected.

---

# 16. Phase 13: Configuration Manager

## Objective

Store persistent device configuration.

### Configuration examples

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

### Storage

Use ESP-IDF NVS.

### Requirements

- Configuration version.
- Default values.
- Validation.
- Factory defaults.
- Safe update.
- Persistent storage.

### Exit Criteria

- Configuration survives reboot.
- Invalid configuration is rejected.
- Defaults can be restored.

---

# 17. Phase 14: Main Device UI

## Objective

Create the complete physical-device user interface.

### Screens

```text
Main
Measurements
Calibration
Configuration
System Status
```

### Main screen

Display:

- Conductivity
- TOF
- Distance
- Measurement status

Optional:

- Battery level
- SD status
- Wi-Fi status

### Navigation

```text
Encoder CW/CCW
        ↓
Select Item

Encoder Press
        ↓
Open Item

BACK
        ↓
Return

START
        ↓
Start Measurement
```

### Exit Criteria

- All screens are navigable.
- START performs measurement.
- Errors are visible.
- UI does not block measurement operations.

---

# 18. Phase 15: Wi-Fi Access Point

## Objective

Provide local wireless access.

### Default configuration

```text
SSID:
EC-TOF-Analyzer

Mode:
Wi-Fi Access Point

Example IP:
192.168.4.1
```

### Tasks

- Initialize Wi-Fi.
- Configure AP.
- Start DHCP.
- Handle client connections.
- Display Wi-Fi status.

### Exit Criteria

- Phone or laptop can connect.
- Device receives a client connection.
- Web server is reachable.

---

# 19. Phase 16: HTTP Server

## Objective

Implement the local web interface.

### Required pages

```text
/
 /dashboard
 /measurements
 /calibration
 /configuration
 /system
```

### REST endpoints

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

### Architecture

```text
Web Request
    ↓
HTTP Handler
    ↓
Application Service
    ↓
Manager
    ↓
Driver
```

The HTTP layer must not directly control hardware drivers.

### Exit Criteria

- Dashboard loads.
- Current measurement is displayed.
- Measurement can be started.
- Stored data can be viewed.
- CSV data can be downloaded.

---

# 20. Phase 17: Web Dashboard

## Objective

Create a simple local browser interface.

### Dashboard

```text
+--------------------------------------+
| EC-TOF ANALYZER                      |
+--------------------------------------+
| Conductivity                         |
| 4.82 mS/cm                           |
|                                      |
| TOF                                  |
| 12.482 us                            |
|                                      |
| Distance                             |
| 18.73 mm                             |
|                                      |
| Status: READY                        |
|                                      |
| [ START MEASUREMENT ]                |
+--------------------------------------+
```

### Additional pages

#### Measurements

Show:

- Timestamp
- Conductivity
- TOF
- Distance
- Status

#### Calibration

Provide controlled calibration operations.

#### Configuration

Provide device configuration.

#### System

Show:

- Firmware version
- Uptime
- SD status
- Wi-Fi status
- RTC status
- EC sensor status
- TOF sensor status
- System errors

---

# 21. Phase 18: Error Handling and System Hardening

## Objective

Make the prototype stable during normal operation.

### Required error conditions

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

### Error behavior

The system should:

1. Detect the error.
2. Record the error.
3. Display a useful message.
4. Prevent invalid measurement logging.
5. Recover automatically where possible.
6. Continue operating other subsystems where safe.

### Example

```text
EC SENSOR ERROR

Unable to read conductivity.

Check:
- Sensor connection
- RS485 connection
- Sensor power

[BACK]
```

---

# 22. Phase 19: Power System Integration

## Objective

Integrate battery operation.

### Power architecture

```text
3.7 V Battery
      |
      +--------------------+
      |                    |
      v                    v
12 V Boost             5 V Regulator
      |                    |
      v                    v
 SEN0707             TFT / HC-SR04
                           |
                           v
                     3.3 V Logic
                           |
                           v
                       ESP32-S3
```

### Tasks

- Integrate battery.
- Integrate USB-C charging.
- Integrate power switch.
- Add fuse/polyfuse.
- Verify regulator output.
- Verify sensor power.
- Verify display power.
- Measure current consumption.
- Test charging.
- Test low-battery behavior.

### Important

Do not assume theoretical battery runtime.

Measure actual prototype consumption.

### Exit Criteria

- Device operates from battery.
- Charging works.
- Power switching works.
- No unstable voltage behavior is observed.

---

# 23. Phase 20: Enclosure Integration

## Objective

Create the first physical prototype enclosure.

### Front panel

Recommended:

```text
+--------------------------------------+
|                                      |
|          4" TFT DISPLAY              |
|                                      |
|                                      |
|                              START   |
|                                      |
|       ROTARY             BACK        |
|       ENCODER                        |
|                                      |
+--------------------------------------+
```

### External connections

Provide removable connectors for:

- EC sensor
- Ultrasonic sensor
- USB-C charging/programming

### Internal considerations

- Battery mounting
- SD card access
- ESP32 mounting
- RS485 module
- Boost converter
- Wiring
- Fuse
- Power switch
- Cable strain relief
- Electrical isolation where needed

### Exit Criteria

- Components fit safely.
- Sensors remain removable.
- No exposed dangerous connections.
- Buttons and display are accessible.

---

# 24. Phase 21: Integrated Testing

## Objective

Verify the complete device.

### Test sequence

```text
Power ON
   ↓
System Initialization
   ↓
Hardware Check
   ↓
READY
   ↓
START
   ↓
TOF Measurement
   ↓
EC Measurement
   ↓
Validation
   ↓
Display
   ↓
Log
   ↓
READY
```

### Basic test

Perform at least:

- 10 consecutive measurements.
- 50 consecutive measurements.
- Reboot test.
- SD removal test.
- EC sensor disconnect test.
- Ultrasonic sensor disconnect test.
- Wi-Fi reconnect test.
- Battery operation test.

---

# 25. Test Matrix

| Test Area | Test | Expected Result |
|---|---|---|
| Boot | Power ON | Device starts normally |
| Display | Display initialization | Screen works |
| Encoder | Rotate | Selection changes |
| Encoder | Press | Selection opens |
| START | Press | Measurement begins |
| BACK | Press | Previous screen opens |
| RTC | Read time | Correct timestamp |
| SD | Insert card | Card mounts |
| SD | Write CSV | Data is stored |
| SD | Remove card | Error handled |
| EC | Read sensor | Conductivity returned |
| EC | Disconnect | Error reported |
| RS485 | Invalid frame | Frame rejected |
| TOF | Valid echo | TOF returned |
| TOF | No echo | Timeout |
| Calibration | Save calibration | Value persists |
| Wi-Fi | Connect phone | AP connection succeeds |
| Web | Open dashboard | Dashboard loads |
| Web | Start measurement | Measurement starts |
| Web | Export CSV | File downloads |
| Power | Battery operation | Device operates |
| Power | USB charging | Battery charges |
| Recovery | Reboot | System recovers |

---

# 26. Milestones

## Milestone 1: Controller Bring-Up

Deliverables:

- ESP32-S3 project
- GPIO testing
- Serial logging
- Basic system startup

---

## Milestone 2: UI Hardware

Deliverables:

- TFT working
- Rotary encoder working
- START button working
- BACK button working

---

## Milestone 3: Storage and Time

Deliverables:

- DS3231 working
- MicroSD working
- CSV logging working

---

## Milestone 4: EC Sensor

Deliverables:

- UART working
- RS485 working
- SEN0707 communication working
- Conductivity readings available

---

## Milestone 5: TOF

Deliverables:

- HC-SR04 working
- TOF measurement working
- Prototype distance calculation working

---

## Milestone 6: Measurement Engine

Deliverables:

- Measurement state machine
- EC + TOF integration
- Measurement validation
- Displayed results
- Logged results

---

## Milestone 7: Calibration

Deliverables:

- EC calibration
- TOF calibration
- NVS persistence

---

## Milestone 8: Web Interface

Deliverables:

- Wi-Fi AP
- HTTP server
- REST API
- Dashboard
- Data export

---

## Milestone 9: Portable Prototype

Deliverables:

- Battery operation
- Charging
- Power switch
- Enclosure
- External sensor connectors

---

## Milestone 10: V0.1 Validation

Deliverables:

- Full system test
- Error recovery test
- Measurement repeatability test
- Data logging validation
- Battery validation
- Documentation

---

# 27. Definition of Done

A feature is considered complete when:

- Code is implemented.
- Code compiles without warnings that affect operation.
- Hardware is tested.
- Normal operation works.
- Error handling is implemented.
- Serial logs are understandable.
- The feature does not break existing functions.
- Relevant test cases pass.
- Configuration is documented.
- The implementation is committed to Git.

---

# 28. V0.1 Acceptance Criteria

The prototype is considered complete when all of the following are functional.

## Measurement

- Conductivity can be measured.
- Ultrasonic TOF can be measured.
- Prototype distance can be calculated.
- Measurements have timestamps.
- Invalid measurements are rejected.

## Display

- Current measurement is visible.
- Device status is visible.
- Errors are visible.
- Physical controls operate correctly.

## Storage

- Measurements can be stored on MicroSD.
- CSV files contain valid data.
- Data survives device reboot.

## RTC

- Measurements contain valid timestamps.
- RTC configuration can be maintained.

## Calibration

- EC calibration is supported.
- TOF calibration is supported.
- Calibration survives reboot.

## Connectivity

- Device can create a local Wi-Fi AP.
- Browser can connect.
- Current measurements can be viewed.
- Measurements can be exported.

## Power

- Device can operate from battery.
- Battery can be charged using USB-C.
- Device can be turned off using a physical switch.

## Reliability

- Sensor disconnection is detected.
- Sensor timeout is detected.
- SD errors are handled.
- The system does not hang during normal sensor failures.

---

# 29. Risk Register

| Risk | Impact | Mitigation |
|---|---|---|
| HC-SR04 unsuitable for final TOF application | High | Keep TOF driver replaceable |
| RS485 noise | High | Proper wiring, grounding and termination |
| SEN0707 power requirement | Medium | Dedicated 12 V boost converter |
| Battery runtime lower than expected | Medium | Measure actual consumption |
| SD card write failure | Medium | Error handling and buffered writes |
| Electrical noise from boost converters | High | Separate power paths and filtering |
| ESP32 GPIO conflicts | Medium | Complete pin mapping before PCB |
| Display SPI conflicts | Medium | Confirm SPI bus architecture early |
| TOF environmental variation | High | Use controlled test geometry |
| Sensor calibration drift | Medium | Store calibration parameters and validate |
| Heat from regulators | Medium | Measure thermal performance |
| Enclosure space | Medium | Prototype physical layout early |
| Wi-Fi affects timing | Medium | Keep measurement timing independent |

---

# 30. Prototype vs Production Boundary

V0.1 must remain a prototype.

The following components or methods are not considered production-final:

### Ultrasonic

HC-SR04 is only a feasibility component.

A final acoustic or ultrasonic transducer system must be evaluated separately.

### Power

The initial battery and boost converter architecture is for prototype validation.

Production hardware may require:

- Dedicated power-management IC
- Battery protection
- Fuel gauge
- Better regulation
- EMI filtering
- Thermal design

### PCB

V0.1 may use development boards and modules.

Production should move toward:

- Custom PCB
- Controlled grounding
- Proper connectorization
- EMI protection
- ESD protection
- Production power architecture

### Enclosure

The first enclosure can be 3D printed.

Production enclosure requirements will be defined after prototype testing.

---

# 31. Recommended Development Order

The implementation order should remain:

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

Do not skip directly to the complete application.

Each subsystem should be validated before the next integration stage.

---

# 32. Git Development Strategy

Use small commits.

Recommended commit structure:

```text
feat: initialize ESP32-S3 project
feat: add GPIO driver
feat: add ST7796 display driver
feat: add input manager
feat: add DS3231 RTC driver
feat: add SD storage manager
feat: add UART driver
feat: add RS485 driver
feat: add SEN0707 driver
feat: add EC measurement
feat: add HC-SR04 driver
feat: add TOF measurement
feat: add measurement manager
feat: add calibration manager
feat: add configuration manager
feat: add Wi-Fi AP
feat: add REST API
feat: add web dashboard
feat: add system error handling
feat: add battery monitoring
test: add integrated measurement tests
test: add system validation tests
```

Avoid large commits containing multiple unrelated subsystems.

---

# 33. Development Rules

The following rules apply throughout implementation.

### Rule 1: Keep drivers independent

Drivers must not contain UI logic.

### Rule 2: Keep UI independent

The display must consume application data.

### Rule 3: Keep web independent

The web server must communicate with application managers.

### Rule 4: Avoid blocking operations

Sensor and storage operations must use controlled timeouts.

### Rule 5: Never wait indefinitely

Every external communication operation requires a timeout.

### Rule 6: Validate sensor data

Never log a measurement simply because a sensor returned a value.

### Rule 7: Protect the filesystem

Do not continuously write unnecessary data to the SD card.

### Rule 8: Keep TOF replaceable

The HC-SR04 implementation must not define the architecture of the final TOF system.

### Rule 9: Keep configuration persistent

Calibration and important configuration must survive reboot.

### Rule 10: Prefer simple solutions

Do not add an abstraction unless it provides a clear benefit.

---

# 34. Future Development

After V0.1 validation, the following can be evaluated.

## Hardware

- Laboratory-grade ultrasonic transducer.
- Improved acoustic receiver.
- Custom analog front-end.
- Higher precision timing hardware.
- Custom PCB.
- Battery fuel gauge.
- Improved power management.
- Industrial connectors.
- ESD protection.
- EMI filtering.

## Software

- More advanced TOF processing.
- Signal quality analysis.
- Multiple measurement profiles.
- Automatic calibration workflows.
- Advanced measurement statistics.
- Device configuration backup.
- Firmware update through web interface.
- Measurement graphs.
- Export formats beyond CSV.

## Connectivity

Potential future options:

- USB data interface.
- Bluetooth.
- Wi-Fi network mode.
- Remote data collection.
- Cloud integration.

These features are outside V0.1.

---

# 35. Final Implementation Architecture

The completed V0.1 firmware should follow this structure:

```text
                    +----------------------+
                    |       UI Layer       |
                    | TFT + Input + Web    |
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
     |       Hardware            |
     | ESP32-S3                  |
     | SEN0707                   |
     | HC-SR04                   |
     | ST7796                    |
     | DS3231                    |
     | MicroSD                   |
     +---------------------------+
```

---

# 36. Final Implementation Goal

The V0.1 implementation is successful when the prototype can perform this complete workflow:

```text
Power ON
   ↓
Initialize Hardware
   ↓
Check Sensors
   ↓
READY
   ↓
User Presses START
   ↓
Trigger Ultrasonic Measurement
   ↓
Measure TOF
   ↓
Calculate Prototype Distance
   ↓
Read SEN0707 Conductivity
   ↓
Validate Results
   ↓
Display Measurement
   ↓
Timestamp Measurement
   ↓
Write CSV to MicroSD
   ↓
Update Web Interface
   ↓
Return to READY
```

The most important architectural requirement is that **the measurement application must not depend directly on the HC-SR04 hardware**.

The TOF sensor must remain replaceable.

This allows V0.1 to prove the overall EC + TOF measurement workflow while leaving room for a more suitable laboratory-grade ultrasonic implementation in the next hardware revision.

---

# 37. V0.1 Implementation Baseline

| Area | V0.1 Implementation |
|---|---|
| MCU | ESP32-S3 |
| Firmware | ESP-IDF + C++ |
| EC Sensor | DFRobot SEN0707 |
| EC Interface | RS485 Modbus RTU |
| RS485 | MAX3485 |
| TOF Sensor | HC-SR04 |
| TOF Purpose | Prototype / feasibility |
| Display | 4" 480x320 ST7796 SPI |
| RTC | DS3231 |
| Storage | MicroSD |
| Input | Rotary encoder + buttons |
| Wireless | ESP32 Wi-Fi AP |
| Web | Local HTTP server |
| Battery | 3.7 V 5000 mAh target |
| EC Power | 12 V boost |
| Charging | USB-C |
| Logging | CSV |
| Configuration | ESP32 NVS |
| Calibration | EC + TOF |
| Architecture | Modular |
| Production Status | Prototype |

---

# 38. Implementation Completion

When all milestones and acceptance criteria are complete, the project can be tagged:

```text
EC-TOF Analyzer V0.1
```

The next development phase should not immediately become V1.0.

Instead, the prototype results should be reviewed to determine:

1. Whether the EC measurement meets the customer's requirement.
2. Whether the TOF measurement concept is technically viable.
3. Whether the HC-SR04 must be replaced.
4. What ultrasonic transducer architecture is required.
5. What accuracy and repeatability can actually be achieved.
6. Whether a custom PCB is justified.
7. Whether the battery architecture is sufficient.
8. What changes are required for a production-oriented V1 design.

Only after these questions are answered should the project move into the next hardware and software revision.