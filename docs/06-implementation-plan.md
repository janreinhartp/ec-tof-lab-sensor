# EC-TOF Analyzer V0.1
## Implementation Plan

**Version:** 0.1  
**Status:** Prototype  
**Target Platform:** ESP32-S3  
**Firmware:** ESP-IDF  
**Last Updated:** October 2026

---

# 1. Purpose

This document defines the implementation plan for the EC-TOF Analyzer V0.1 prototype.

The implementation will follow an incremental approach.

Each phase should produce a testable result before moving to the next phase.

The primary objective is to prove the complete measurement workflow:

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

---

# 2. Implementation Strategy

The project shall be developed in layers.

Recommended order:

```text
Hardware Bring-Up
       │
       ▼
ESP32-S3 Base
       │
       ▼
Drivers
       │
       ▼
Individual Sensors
       │
       ▼
Measurement Logic
       │
       ▼
Display + Controls
       │
       ▼
Storage + RTC
       │
       ▼
Battery Indicator
       │
       ▼
Wi-Fi + Web UI
       │
       ▼
Calibration
       │
       ▼
Integration Testing
       │
       ▼
Prototype Validation
```

The system shall not attempt to implement all features simultaneously.

---

# 3. Phase 0 - Project Preparation

## Objectives

Set up the development environment.

## Tasks

- Install ESP-IDF.
- Configure VS Code.
- Configure ESP-IDF extension.
- Create project repository.
- Configure ESP32-S3 target.
- Configure build system.
- Configure serial monitor.
- Create initial project structure.
- Create Git repository.
- Establish coding conventions.

## Expected Result

The ESP32-S3 shall build and flash a basic application.

Example:

```text
EC-TOF Analyzer V0.1
Booting...
ESP32-S3 Ready
```

---

# 4. Phase 1 - ESP32-S3 Hardware Bring-Up

## Objectives

Verify the controller board.

## Tasks

- Verify power.
- Flash firmware.
- Verify serial output.
- Verify reset behavior.
- Verify GPIO operation.
- Verify basic timer operation.
- Verify Wi-Fi hardware availability.
- Verify SPI peripheral.
- Verify I2C peripheral.
- Verify UART peripheral.
- Verify ADC peripheral.

## Expected Result

All required ESP32-S3 peripherals can be initialized successfully.

---

# 5. Phase 2 - Project Structure

Create the initial software architecture.

```text
main/
    main.cpp

    app/
    sensors/
        ec/
        tof/

    drivers/
        rs485/
        uart/
        spi/
        i2c/
        gpio/
        adc/
        timer/

    display/
    input/
    storage/
    rtc/
    web/
    power/
    system/
    common/
```

## Tasks

- Create modules.
- Create interfaces.
- Create common data structures.
- Create logging utilities.
- Create error definitions.
- Create configuration structures.

## Expected Result

The project builds with the complete basic architecture.

---

# 6. Phase 3 - GPIO Driver

## Objectives

Create the GPIO abstraction.

## Tasks

Implement:

- Digital output
- Digital input
- Pull-up configuration
- Pull-down configuration
- Interrupt support where required

## Expected Result

Application code can use GPIO without directly accessing ESP-IDF GPIO implementation details.

---

# 7. Phase 4 - UART Driver

## Objectives

Create the UART abstraction required by the RS485 interface.

## Tasks

Implement:

- UART initialization
- Baud rate configuration
- TX
- RX
- Timeout
- Buffer handling
- UART error handling

## Expected Result

The ESP32-S3 can communicate through a UART peripheral.

---

# 8. Phase 5 - RS485 Driver

## Objectives

Implement RS485 communication.

Hardware:

```text
ESP32-S3 UART
      │
      ▼
MAX3485
      │
      ▼
RS485
```

## Tasks

Implement:

- RS485 initialization
- Driver enable control
- Transmit
- Receive
- Receive timeout
- Direction control
- Frame handling

## Expected Result

Reliable RS485 communication is available to the application.

---

# 9. Phase 6 - SEN0707 Driver

## Objectives

Communicate with the conductivity sensor.

## Tasks

Implement:

- SEN0707 initialization
- Modbus RTU frame creation
- Modbus request transmission
- Response reception
- CRC validation
- Register decoding
- Conductivity conversion
- Communication timeout
- Sensor error detection

## Expected Result

The firmware can read conductivity from the SEN0707.

Example:

```text
EC: 4820 µS/cm
Status: VALID
```

---

# 10. Phase 7 - EC Sensor Service

Create the EC sensor abstraction.

Example:

```cpp
class IEcSensor
{
public:
    virtual bool begin() = 0;
    virtual bool read(float& conductivity) = 0;
    virtual bool calibrate() = 0;
    virtual bool isConnected() = 0;

    virtual ~IEcSensor() = default;
};
```

## Tasks

- Create `IEcSensor`.
- Implement `Sen0707Driver`.
- Create EC Manager.
- Add error handling.
- Add measurement validation.

## Expected Result

The application can request an EC measurement without knowing the underlying Modbus implementation.

---

# 11. Phase 8 - Ultrasonic GPIO Driver

## Objectives

Prepare the hardware interface for the HC-SR04.

## Tasks

Implement:

- Trigger output
- Echo input
- Timer capture
- Timeout
- Echo validation

The ECHO signal shall pass through an appropriate voltage divider or level shifter.

## Expected Result

The ESP32-S3 can safely trigger the HC-SR04 and measure the echo pulse.

---

# 12. Phase 9 - TOF Sensor Driver

## Objectives

Implement the HC-SR04 prototype driver.

## Tasks

Implement:

- Trigger pulse
- Echo timing
- TOF calculation
- Timeout handling
- Invalid measurement detection

## Expected Result

The firmware can produce repeatable TOF values.

Example:

```text
TOF: 12.482 us
```

---

# 13. Phase 10 - TOF Sensor Abstraction

Create:

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

## Expected Result

The measurement system is independent of the HC-SR04 implementation.

---

# 14. Phase 11 - TOF Distance Calculation

For the V0.1 air prototype:

```text
Distance = TOF × Speed of Sound / 2
```

## Tasks

- Implement distance calculation.
- Define units.
- Define configurable speed of sound.
- Add validation limits.
- Add invalid result handling.

## Expected Result

The system produces a calculated distance.

---

# 15. Phase 12 - TFT Display Driver

## Objectives

Bring up the 4-inch ST7796 display.

## Tasks

- Configure SPI.
- Initialize ST7796.
- Test screen orientation.
- Test screen clearing.
- Test text rendering.
- Test basic graphics.
- Test screen refresh.

## Expected Result

The display shows:

```text
EC-TOF ANALYZER
System Ready
```

---

# 16. Phase 13 - Display Manager

Create the application-level display system.

## Screens

Implement:

1. Main
2. Measurement
3. Calibration
4. Configuration
5. System Status

The display manager shall receive application data instead of directly accessing sensors.

## Expected Result

The application can switch between UI screens.

---

# 17. Phase 14 - Physical Input

## Hardware

- Rotary encoder
- Encoder push button
- START
- BACK

## Tasks

Implement:

- GPIO input
- Debouncing
- Encoder direction detection
- Encoder press
- START event
- BACK event

Create logical events:

```text
ENCODER_CW
ENCODER_CCW
ENCODER_PRESS
START_PRESS
BACK_PRESS
```

## Expected Result

The application can navigate menus and start measurements.

---

# 18. Phase 15 - Measurement Manager

## Objectives

Implement the complete measurement state machine.

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

## Tasks

- Create measurement state enum.
- Implement state transitions.
- Implement TOF execution.
- Implement EC execution.
- Implement validation.
- Create measurement record.
- Update UI.
- Trigger storage.

## Expected Result

Pressing START performs a complete measurement.

---

# 19. Phase 16 - RTC Integration

## Hardware

DS3231.

## Interface

I2C.

## Tasks

- Implement I2C driver.
- Implement DS3231 driver.
- Read date/time.
- Set date/time.
- Validate RTC.
- Create timestamp service.

## Expected Result

The system can produce timestamps.

Example:

```text
2026-10-08T15:32:10
```

---

# 20. Phase 17 - MicroSD Integration

## Hardware

- MicroSD module
- SPI interface

## Tasks

- Initialize SPI.
- Mount filesystem.
- Detect SD card.
- Create directories.
- Create CSV files.
- Append records.
- Handle write failures.
- Unmount safely.

## Expected Result

The system can store measurements.

---

# 21. Phase 18 - Measurement Logging

Implement the final measurement record:

```text
timestamp,conductivity,tof_us,distance_mm,status
```

Example:

```text
2026-10-08T15:32:10,4820,12.482,18.73,VALID
```

## Tasks

- Generate CSV row.
- Add timestamp.
- Write record.
- Verify write.
- Handle SD errors.

## Expected Result

Every valid measurement can be saved to the SD card.

---

# 22. Phase 19 - Battery Hardware

## Hardware

- 3.7 V rechargeable battery
- USB-C charger
- Battery protection
- Voltage divider
- Appropriate power regulators

Power architecture:

```text
USB-C
   │
   ▼
Charger
   │
   ▼
Battery
   │
   ▼
Power Distribution
   ├── 12 V ──► SEN0707
   ├── 5 V ───► TFT / HC-SR04
   └── 3.3 V ─► ESP32-S3 / Logic
```

## Expected Result

The complete system can operate from the battery.

---

# 23. Phase 20 - Battery Voltage Measurement

## Objectives

Measure battery voltage using the ESP32-S3 ADC.

Architecture:

```text
Battery
   │
   ▼
Voltage Divider
   │
   ▼
ESP32 ADC
   │
   ▼
Battery Voltage
```

## Tasks

- Configure ADC.
- Implement voltage divider calculation.
- Calibrate ADC if required.
- Add voltage filtering.
- Validate voltage range.

## Expected Result

The system can display the measured battery voltage.

Example:

```text
Battery: 3.82 V
```

---

# 24. Phase 21 - Battery Percentage and Indicator

## Objectives

Create a simple battery indicator.

## Tasks

- Convert battery voltage to estimated percentage.
- Implement lookup table.
- Implement battery state.
- Add low battery warning.
- Add battery icon to TFT.
- Add battery information to web API.

Example:

```text
🔋 78%
```

Example status:

```text
Battery: 3.82 V
Level:   50%
Status:  NORMAL
```

The percentage is an estimate.

No battery current measurement shall be implemented.

No battery power calculation shall be implemented.

No dedicated fuel gauge is required for V0.1.

## Expected Result

The user can quickly determine the approximate battery level.

---

# 25. Phase 22 - Configuration Manager

## Objectives

Implement persistent configuration using ESP-IDF NVS.

## Configuration

Potential settings:

- Device settings
- Sensor settings
- TOF settings
- Calibration values
- Display settings
- Logging settings

## Tasks

- Create configuration structure.
- Implement defaults.
- Implement NVS storage.
- Implement load.
- Implement save.
- Implement validation.
- Add configuration version.

## Expected Result

Configuration survives a reboot.

---

# 26. Phase 23 - Calibration

## EC Calibration

Implement the calibration workflow for the SEN0707.

Tasks:

- Calibration screen.
- Calibration value input.
- Sensor reading.
- Calibration validation.
- Save calibration parameters.

## TOF Calibration

Tasks:

- Reference distance entry.
- Measurement collection.
- Correction calculation.
- Save calibration parameters.

## Expected Result

The user can perform basic prototype calibration without modifying firmware code.

---

# 27. Phase 24 - Wi-Fi Access Point

## Objectives

Enable local wireless access.

Recommended:

```text
SSID: EC-TOF-Analyzer
IP:   192.168.4.1
```

## Tasks

- Initialize Wi-Fi.
- Start access point.
- Configure IP.
- Handle client connections.
- Handle Wi-Fi errors.

## Expected Result

A phone, tablet, or computer can connect directly to the analyzer.

---

# 28. Phase 25 - REST API

Implement:

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

## Expected Result

The analyzer exposes measurement and system data through HTTP.

---

# 29. Phase 26 - Web UI

## Pages

Implement:

### Dashboard

Display:

- EC
- TOF
- Distance
- Status
- Battery

### Measurements

Display:

- Measurement history
- Timestamp
- EC
- TOF
- Distance
- Status

### Calibration

Provide calibration controls.

### Configuration

Provide configuration controls.

### System Status

Display:

- Firmware version
- Uptime
- RTC status
- SD status
- Sensor status
- Battery voltage
- Battery percentage

## Expected Result

The complete device can be monitored and controlled from a local browser.

---

# 30. Phase 27 - Integrated Measurement Workflow

At this stage all major subsystems shall be connected.

Complete flow:

```text
User
 │
 ▼
START
 │
 ▼
Measurement Manager
 │
 ├──► TOF
 │      │
 │      ▼
 │   Distance
 │
 └──► EC
        │
        ▼
   Conductivity
        │
        ▼
    Validation
        │
        ├──► TFT
        │
        ├──► Web
        │
        └──► SD
```

## Expected Result

One START action produces a complete measurement record.

---

# 31. Phase 28 - Error Handling

Implement consistent error handling.

## EC Errors

- Sensor disconnected
- RS485 timeout
- CRC failure
- Invalid Modbus response
- UART failure

## TOF Errors

- No echo
- Timeout
- Invalid pulse
- Out-of-range distance

## SD Errors

- Card missing
- Mount failure
- Write failure
- Filesystem error

## RTC Errors

- Communication failure
- Invalid time

## Battery Errors

- ADC failure
- Invalid voltage
- Low battery

## Expected Result

Failures are reported clearly without crashing the system.

---

# 32. Phase 29 - System Status

Create a system health summary.

Example:

```text
SYSTEM STATUS

EC Sensor:     OK
TOF Sensor:    OK
RTC:           OK
SD Card:       OK
Wi-Fi:         OK

Battery:       3.82 V
Battery:       52%
Battery State: NORMAL
```

The same information shall be available through the web interface.

---

# 33. Phase 30 - Prototype Enclosure

## Objectives

Integrate the electronics into the physical prototype.

## Tasks

- Design enclosure.
- Mount TFT.
- Mount rotary encoder.
- Mount START button.
- Mount BACK button.
- Mount power switch.
- Add USB-C access.
- Add sensor connectors.
- Add SD access if required.
- Provide battery compartment.
- Provide strain relief.
- Provide ventilation where required.

External sensors shall remain removable.

---

# 34. Phase 31 - Hardware Integration Testing

Test every subsystem independently.

## EC

- Sensor connection
- Sensor disconnection
- Conductivity readings
- RS485 communication
- CRC handling
- Timeout handling

## TOF

- Trigger
- Echo
- Timing
- Distance calculation
- Timeout

## Display

- Startup
- Screen switching
- Measurement display
- Error display
- Battery indicator

## Input

- Encoder
- Encoder press
- START
- BACK

## SD

- Mount
- File creation
- Logging
- Removal
- Write failure

## RTC

- Read
- Write
- Timestamp generation

## Battery

- ADC voltage
- Percentage estimation
- Low battery indication
- Charging behavior

## Wi-Fi

- Access point
- Connection
- Dashboard
- REST API

---

# 35. Phase 32 - Measurement Repeatability Testing

The prototype shall be tested using repeatable reference conditions.

## EC Testing

Measure the same sample multiple times.

Record:

- Minimum
- Maximum
- Average
- Variation

## TOF Testing

Measure known distances repeatedly.

Record:

- Reference distance
- Measured distance
- Error
- Repeatability

The purpose is to determine whether the prototype measurement concept is viable.

---

# 36. Phase 33 - Long-Run Testing

The system shall operate continuously for an extended period.

Test:

- Sensor communication
- SD logging
- Display operation
- Wi-Fi
- RTC
- Battery operation
- Memory stability
- Task stability

Monitor for:

- Crashes
- Watchdog resets
- Memory leaks
- Sensor communication failures
- SD corruption
- Unexpected reboots

---

# 37. Phase 34 - Battery Runtime Test

Battery runtime shall be tested under representative operating conditions.

Record:

- Starting battery voltage
- Starting estimated percentage
- Test duration
- Measurement frequency
- Display usage
- Wi-Fi usage
- Ending battery voltage
- Ending estimated percentage

The test shall evaluate practical runtime.

Battery current measurement is not required.

---

# 38. Phase 35 - Final V0.1 Validation

Perform the complete workflow.

```text
Power On
   │
   ▼
System Initialization
   │
   ▼
Battery Indicator
   │
   ▼
Sensor Check
   │
   ▼
READY
   │
   ▼
START
   │
   ▼
TOF Measurement
   │
   ▼
Distance Calculation
   │
   ▼
EC Measurement
   │
   ▼
Validation
   │
   ▼
Display
   │
   ▼
SD Logging
   │
   ▼
Web Update
   │
   ▼
READY
```

---

# 39. Development Milestones

## Milestone 1 - Controller

- ESP32-S3 boots
- Project builds
- Basic drivers work

## Milestone 2 - EC

- RS485 works
- SEN0707 responds
- Conductivity is readable

## Milestone 3 - TOF

- HC-SR04 works
- TOF is measured
- Distance is calculated

## Milestone 4 - UI

- TFT works
- Encoder works
- Buttons work

## Milestone 5 - Measurement

- Complete measurement state machine works

## Milestone 6 - Data

- RTC works
- SD logging works

## Milestone 7 - Battery

- Battery voltage works
- Battery percentage works
- Battery indicator works

## Milestone 8 - Web

- Wi-Fi works
- REST API works
- Web dashboard works

## Milestone 9 - Calibration

- EC calibration works
- TOF calibration works

## Milestone 10 - Integrated Prototype

- Complete system works from battery
- Measurements are repeatable
- Data is logged
- Web interface works
- Physical controls work

---

# 40. Definition of Done

V0.1 shall be considered complete when all of the following are satisfied.

## Hardware

- ESP32-S3 installed.
- SEN0707 connected through RS485.
- HC-SR04 connected safely.
- TFT connected.
- MicroSD connected.
- DS3231 connected.
- Rotary encoder connected.
- START button connected.
- BACK button connected.
- Battery installed.
- USB-C charging works.
- Power rails operate correctly.
- External sensors are removable.

## Firmware

- ESP-IDF project builds.
- Firmware boots reliably.
- Sensor drivers operate.
- Measurement state machine operates.
- EC measurement works.
- TOF measurement works.
- Distance calculation works.
- RTC timestamps work.
- SD logging works.
- Configuration persists.
- Calibration persists.
- Battery voltage is measured.
- Battery indicator works.
- Wi-Fi works.
- REST API works.
- Web UI works.
- Error handling works.

## Validation

- EC repeatability tested.
- TOF repeatability tested.
- SD logging tested.
- Battery operation tested.
- Wi-Fi tested.
- Long-run operation tested.
- Complete measurement workflow tested.

---

# 41. Prototype Limitations

V0.1 is a development prototype.

The following limitations shall be documented:

### Conductivity

The SEN0707 is intended for the defined conductivity range and sample type.

Final application-specific validation is required.

### Ultrasonic

The HC-SR04 is a low-cost air ultrasonic module.

It is not the final laboratory-grade acoustic measurement system.

The V0.1 TOF implementation therefore validates the measurement architecture rather than final liquid/acoustic measurement performance.

### Battery

Battery percentage is estimated from voltage.

It is not a precision state-of-charge measurement.

### Enclosure

The prototype enclosure may not provide final environmental protection.

### Accuracy

V0.1 accuracy shall not be considered production accuracy.

---

# 42. Future Improvements

Potential future improvements include:

- Production-grade ultrasonic transducer
- Dedicated ultrasonic receiver
- Higher-precision TOF timing
- Improved acoustic measurement method
- Improved conductivity sensor
- Dedicated battery fuel gauge
- Advanced battery management
- USB data export
- OTA firmware update
- Cloud synchronization
- Measurement history analytics
- User authentication
- Improved enclosure
- Waterproof connectors
- Production PCB
- EMC improvements
- Factory calibration workflow

These features should only be added after the V0.1 measurement concept has been validated.

---

# 43. Final Development Order

The recommended final development order is:

```text
01. ESP32-S3 Bring-Up
        │
02. Project Architecture
        │
03. GPIO
        │
04. UART
        │
05. RS485
        │
06. SEN0707
        │
07. EC Manager
        │
08. HC-SR04 GPIO
        │
09. TOF Driver
        │
10. TOF Abstraction
        │
11. Distance Calculation
        │
12. TFT Driver
        │
13. Display Manager
        │
14. Physical Input
        │
15. Measurement Manager
        │
16. RTC
        │
17. MicroSD
        │
18. Data Logging
        │
19. Battery Hardware
        │
20. Battery Voltage
        │
21. Battery Indicator
        │
22. Configuration
        │
23. Calibration
        │
24. Wi-Fi
        │
25. REST API
        │
26. Web UI
        │
27. Integrated Measurement
        │
28. Error Handling
        │
29. System Status
        │
30. Enclosure
        │
31. Hardware Testing
        │
32. Repeatability Testing
        │
33. Long-Run Testing
        │
34. Battery Runtime Testing
        │
35. Final V0.1 Validation
```

---

# 44. Final V0.1 Goal

The final prototype shall provide a portable instrument capable of performing the following workflow:

```text
┌──────────────────────────────┐
│       EC-TOF ANALYZER        │
│                              │
│  Battery: 🔋 78%             │
│                              │
│  Conductivity: 4.82 mS/cm   │
│  TOF:          12.482 us     │
│  Distance:     18.73 mm      │
│                              │
│  Status: READY                │
└──────────────────────────────┘
```

The user presses START.

The analyzer:

1. Triggers the ultrasonic measurement.
2. Measures TOF.
3. Calculates distance.
4. Reads conductivity.
5. Validates the result.
6. Displays the result.
7. Adds a timestamp.
8. Saves the result to MicroSD.
9. Makes the result available through the local web interface.
10. Returns to READY.

The V0.1 implementation should prioritize proving this complete workflow before adding advanced features.