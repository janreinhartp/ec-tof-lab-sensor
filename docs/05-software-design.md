# EC-TOF Analyzer V0.1
# Software Design

## 1. Purpose

This document defines the software architecture and implementation design for the EC-TOF Analyzer V0.1 firmware.

The firmware is responsible for:

- Conductivity measurement.
- Ultrasonic TOF measurement.
- Distance calculation.
- Measurement sequencing.
- Measurement validation.
- Display management.
- Physical user input.
- RTC timestamping.
- MicroSD data logging.
- Battery voltage measurement.
- Simple battery percentage estimation.
- Local Wi-Fi access point.
- REST API.
- Configuration storage.
- Calibration management.
- System diagnostics.
- Error handling.

The firmware will use **ESP-IDF with C++**.

---

# 2. Software Design Principles

The firmware should follow these principles:

1. Keep hardware drivers independent from application logic.
2. Use interfaces for replaceable sensors.
3. Keep the measurement sequence deterministic.
4. Keep UI code separate from sensor code.
5. Keep web API code separate from hardware drivers.
6. Avoid unnecessary FreeRTOS tasks.
7. Avoid blocking operations in time-sensitive measurement code.
8. Make configuration persistent.
9. Make sensor failures recoverable where possible.
10. Keep V0.1 simple enough to debug on real hardware.
11. Design the TOF subsystem so the HC-SR04 can be replaced later.
12. Do not make the production ultrasonic design a dependency of V0.1.
13. Do not implement battery current or power measurement.

---

# 3. Technology Stack

| Component | Technology |
|---|---|
| MCU | ESP32-S3 |
| Framework | ESP-IDF |
| Language | C++ |
| UI | LVGL |
| Display | ST7796 SPI |
| EC communication | Modbus RTU |
| EC physical interface | RS485 |
| TOF | HC-SR04 V0.1 |
| Storage | FAT filesystem on MicroSD |
| RTC | DS3231 |
| Configuration | ESP-IDF NVS |
| Network | ESP32 Wi-Fi AP |
| Web server | ESP-IDF HTTP Server |
| API | REST/JSON |
| Logging | ESP-IDF logging + SD logs |
| Build | ESP-IDF / CMake |

---

# 4. Software Architecture

The firmware uses layered architecture.

```text
┌──────────────────────────────────────────┐
│              User Interface              │
│                                          │
│       LVGL TFT UI / Web UI               │
└────────────────────┬─────────────────────┘
                     │
┌────────────────────▼─────────────────────┐
│           Application Layer              │
│                                          │
│ Measurement Manager                      │
│ Configuration Manager                    │
│ Calibration Manager                      │
│ System Manager                           │
│ Power Manager                            │
└────────────────────┬─────────────────────┘
                     │
┌────────────────────▼─────────────────────┐
│             Service Layer                │
│                                          │
│ EC Manager                               │
│ TOF Manager                              │
│ Storage Manager                          │
│ RTC Manager                              │
│ Wi-Fi Manager                            │
│ Web/API Manager                          │
└────────────────────┬─────────────────────┘
                     │
┌────────────────────▼─────────────────────┐
│             Driver Layer                 │
│                                          │
│ SEN0707 / Modbus                         │
│ HC-SR04                                  │
│ RS485                                    │
│ SPI                                      │
│ I2C                                      │
│ GPIO                                     │
│ ADC                                      │
│ TFT                                      │
│ MicroSD                                  │
│ DS3231                                   │
└──────────────────────────────────────────┘
```

Upper layers must not directly manipulate low-level hardware when a suitable service or driver exists.

---

# 5. Project Structure

Recommended project structure:

```text
ec-tof-analyzer/
│
├── CMakeLists.txt
├── sdkconfig.defaults
├── partitions.csv
├── README.md
│
├── main/
│   ├── CMakeLists.txt
│   ├── main.cpp
│   │
│   ├── app/
│   │   ├── app_manager.cpp
│   │   ├── app_manager.h
│   │   ├── measurement_manager.cpp
│   │   ├── measurement_manager.h
│   │   ├── calibration_manager.cpp
│   │   ├── calibration_manager.h
│   │   ├── configuration_manager.cpp
│   │   ├── configuration_manager.h
│   │   ├── system_manager.cpp
│   │   └── system_manager.h
│   │
│   ├── sensors/
│   │   ├── ec/
│   │   │   ├── iec_sensor.h
│   │   │   ├── ec_manager.cpp
│   │   │   ├── ec_manager.h
│   │   │   ├── sen0707.cpp
│   │   │   └── sen0707.h
│   │   │
│   │   └── tof/
│   │       ├── itof_sensor.h
│   │       ├── tof_manager.cpp
│   │       ├── tof_manager.h
│   │       ├── hc_sr04.cpp
│   │       └── hc_sr04.h
│   │
│   ├── drivers/
│   │   ├── rs485/
│   │   ├── uart/
│   │   ├── spi/
│   │   ├── i2c/
│   │   ├── gpio/
│   │   └── adc/
│   │
│   ├── display/
│   │   ├── display_manager.cpp
│   │   ├── display_manager.h
│   │   ├── ui_manager.cpp
│   │   └── ui_manager.h
│   │
│   ├── input/
│   │   ├── encoder.cpp
│   │   ├── encoder.h
│   │   ├── buttons.cpp
│   │   └── buttons.h
│   │
│   ├── storage/
│   │   ├── storage_manager.cpp
│   │   ├── storage_manager.h
│   │   ├── csv_logger.cpp
│   │   └── csv_logger.h
│   │
│   ├── rtc/
│   │   ├── rtc_manager.cpp
│   │   └── rtc_manager.h
│   │
│   ├── web/
│   │   ├── wifi_manager.cpp
│   │   ├── wifi_manager.h
│   │   ├── web_server.cpp
│   │   ├── web_server.h
│   │   ├── api_handlers.cpp
│   │   └── api_handlers.h
│   │
│   ├── power/
│   │   ├── power_manager.cpp
│   │   └── power_manager.h
│   │
│   ├── system/
│   │   ├── error_manager.cpp
│   │   ├── error_manager.h
│   │   ├── diagnostics.cpp
│   │   └── diagnostics.h
│   │
│   └── common/
│       ├── types.h
│       ├── constants.h
│       ├── config.h
│       └── result.h
│
└── components/
```

The exact directory structure can be simplified during implementation if a module is small.

---

# 6. Application Startup

The main entry point should remain small.

Example:

```cpp
extern "C" void app_main()
{
    AppManager app;

    app.initialize();
    app.start();
}
```

`AppManager` coordinates system initialization.

The main application should not contain sensor implementation code.

---

# 7. Application Initialization

Recommended startup order:

```text
Boot
 │
 ▼
ESP-IDF Initialization
 │
 ▼
Load Configuration
 │
 ▼
Initialize GPIO
 │
 ▼
Initialize Power Manager
 │
 ▼
Initialize RTC
 │
 ▼
Initialize SPI
 │
 ├── TFT
 └── MicroSD
 │
 ▼
Initialize UART
 │
 ▼
Initialize RS485
 │
 ▼
Initialize EC Sensor
 │
 ▼
Initialize TOF
 │
 ▼
Initialize Input
 │
 ▼
Initialize Display/UI
 │
 ▼
Initialize Wi-Fi
 │
 ▼
Initialize Web Server
 │
 ▼
System Ready
```

Initialization failures should be reported to the system status manager.

---

# 8. Common Data Types

Shared application structures should be defined in `common/types.h`.

## 8.1 Measurement Data

```cpp
struct MeasurementData
{
    uint64_t timestamp;

    float conductivityUsCm;

    float tofUs;

    float distanceMm;

    bool valid;

    uint32_t sequenceNumber;
};
```

---

# 9. Measurement Status

```cpp
enum class MeasurementStatus
{
    IDLE,
    STARTING,
    MEASURING_TOF,
    READING_EC,
    VALIDATING,
    COMPLETE,
    INVALID,
    TIMEOUT,
    SENSOR_ERROR
};
```

---

# 10. System Status

```cpp
enum class SystemStatus
{
    BOOTING,
    INITIALIZING,
    READY,
    MEASURING,
    WARNING,
    ERROR
};
```

---

# 11. Battery Status

V0.1 uses voltage-only battery estimation.

```cpp
enum class BatteryState
{
    NORMAL,
    LOW,
    CRITICAL
};

struct PowerStatus
{
    float batteryVoltage;
    uint8_t batteryPercent;
    BatteryState state;
};
```

There is intentionally no battery current or battery power field.

---

# 12. EC Sensor Interface

The EC subsystem must use an interface so the sensor implementation can be replaced later.

## 12.1 Interface

```cpp
class IEcSensor
{
public:
    virtual ~IEcSensor() = default;

    virtual bool initialize() = 0;

    virtual bool readConductivity(float& conductivityUsCm) = 0;

    virtual bool isConnected() = 0;

    virtual void reset() = 0;
};
```

---

# 13. SEN0707 Driver

The SEN0707 implementation will use:

```text
ESP32 UART
    │
    ▼
RS485 Driver
    │
    ▼
Modbus RTU
    │
    ▼
SEN0707
```

The driver is responsible for:

- Modbus request generation.
- UART transmission.
- Response reception.
- CRC validation.
- Register parsing.
- Timeout handling.
- Sensor error handling.

The driver should not update the TFT or write to the SD card.

---

# 14. Modbus Driver

The Modbus layer should provide:

```cpp
class ModbusRtu
{
public:
    bool initialize();

    bool readHoldingRegisters(
        uint8_t slaveId,
        uint16_t address,
        uint16_t count,
        uint16_t* data
    );

    bool writeHoldingRegister(
        uint8_t slaveId,
        uint16_t address,
        uint16_t value
    );
};
```

The exact register map must follow the SEN0707 documentation.

---

# 15. RS485 Driver

The RS485 driver controls:

- UART.
- DE/RE direction.
- Transmission timing.
- Reception timing.

Example interface:

```cpp
class Rs485Driver
{
public:
    bool initialize();

    bool transmit(
        const uint8_t* data,
        size_t length
    );

    bool receive(
        uint8_t* buffer,
        size_t bufferSize,
        size_t& received
    );
};
```

The RS485 driver must not know about conductivity.

---

# 16. TOF Sensor Interface

The TOF subsystem must be replaceable.

```cpp
class ITofSensor
{
public:
    virtual ~ITofSensor() = default;

    virtual bool initialize() = 0;

    virtual bool measureTof(float& tofUs) = 0;

    virtual bool isConnected() = 0;

    virtual void reset() = 0;
};
```

The HC-SR04 is one implementation of this interface.

---

# 17. HC-SR04 Driver

The HC-SR04 driver is responsible for:

- Trigger pulse.
- Echo detection.
- Microsecond timing.
- Timeout detection.
- TOF calculation.

It must not directly calculate application-level results.

The driver returns the measured TOF.

---

# 18. TOF Distance Calculation

For the V0.1 air prototype:

```text
Distance = TOF × Speed of Sound / 2
```

The software should isolate this calculation from the sensor driver.

Example:

```cpp
float calculateDistanceMm(
    float tofUs,
    float speedOfSoundMps
);
```

The default speed of sound may be configurable.

The software must not assume that the HC-SR04 air calculation is valid for a future submerged acoustic measurement system.

---

# 19. TOF Manager

The `TofManager` coordinates the sensor and application-level processing.

Responsibilities:

- Start TOF measurement.
- Receive raw TOF.
- Validate TOF.
- Calculate distance.
- Report errors.

Example:

```cpp
struct TofMeasurement
{
    float tofUs;
    float distanceMm;
    bool valid;
};
```

---

# 20. Measurement Manager

The Measurement Manager is the core application component.

It controls the complete measurement sequence.

### Responsibilities

- Start measurement.
- Coordinate TOF.
- Coordinate EC.
- Validate results.
- Create measurement records.
- Update system state.
- Notify display.
- Notify storage.
- Notify web clients.

The Measurement Manager must not directly access GPIO, SPI, or UART drivers.

---

# 21. Measurement State Machine

```text
             ┌───────────┐
             │   IDLE    │
             └─────┬─────┘
                   │ START
                   ▼
             ┌───────────┐
             │  START    │
             └─────┬─────┘
                   ▼
          ┌────────────────┐
          │  TOF_TRIGGER   │
          └───────┬────────┘
                  ▼
          ┌────────────────┐
          │    TOF_WAIT    │
          └───────┬────────┘
                  ▼
          ┌────────────────┐
          │  TOF_PROCESS   │
          └───────┬────────┘
                  ▼
          ┌────────────────┐
          │    EC_READ     │
          └───────┬────────┘
                  ▼
          ┌────────────────┐
          │   VALIDATE     │
          └───────┬────────┘
                  ▼
          ┌────────────────┐
          │    DISPLAY     │
          └───────┬────────┘
                  ▼
          ┌────────────────┐
          │      LOG       │
          └───────┬────────┘
                  ▼
             ┌───────────┐
             │  READY    │
             └───────────┘
```

Error paths must return the application to a safe state.

---

# 22. Measurement Validation

A measurement is valid only when:

- TOF measurement completed.
- TOF is within configured limits.
- Distance is within configured limits.
- EC sensor responded successfully.
- Conductivity is within configured limits.
- Required timestamp is available.

Example:

```cpp
bool validateMeasurement(
    const MeasurementData& measurement
);
```

Invalid measurements should not be logged as normal valid results.

They may still be recorded in diagnostic logs.

---

# 23. Measurement Sequence Timing

The firmware should use timeouts for every external operation.

Examples:

| Operation | Timeout |
|---|---:|
| HC-SR04 echo | Configurable |
| RS485 response | Configurable |
| SD write | Configurable |
| RTC transaction | Configurable |
| Wi-Fi operation | Configurable |

No external device should be allowed to block the main application indefinitely.

---

# 24. Display Architecture

The display subsystem is divided into:

```text
Display Manager
      │
      ▼
UI Manager
      │
      ▼
LVGL
      │
      ▼
ST7796 Driver
```

The display should consume application data.

It should not directly query:

- SEN0707.
- HC-SR04.
- DS3231.
- MicroSD.

---

# 25. Display Screens

V0.1 should provide the following screens.

## Main Screen

Shows:

- Conductivity.
- TOF.
- Distance.
- Measurement status.
- Battery percentage.

## Measurement Screen

Shows:

- Current measurement.
- Measurement sequence.
- Sensor status.

## Calibration Screen

Provides access to:

- EC calibration.
- TOF calibration.

## Configuration Screen

Provides:

- Sensor settings.
- Measurement limits.
- Speed of sound.
- Device settings.

## System Status Screen

Shows:

- Firmware version.
- RTC status.
- SD status.
- EC sensor status.
- TOF status.
- Wi-Fi status.
- Battery status.

---

# 26. Input Architecture

The input system abstracts physical controls.

```text
Rotary Encoder
      │
      ▼
Input Manager
      │
      ├── Rotate
      ├── Press
      │
START Button
      │
      ├── Press
      │
BACK Button
      │
      └── Press
```

The UI layer consumes input events rather than directly reading GPIO.

---

# 27. Input Events

Example:

```cpp
enum class InputEvent
{
    NONE,
    ENCODER_CW,
    ENCODER_CCW,
    ENCODER_PRESS,
    START_PRESS,
    BACK_PRESS
};
```

---

# 28. RTC Manager

The RTC Manager abstracts the DS3231.

Example:

```cpp
class RtcManager
{
public:
    bool initialize();

    bool getDateTime(DateTime& time);

    bool setDateTime(const DateTime& time);

    bool isValid();
};
```

The rest of the application should not directly access the I2C driver.

---

# 29. Timestamp Format

Application timestamps should use:

```text
YYYY-MM-DDTHH:MM:SS
```

Example:

```text
2026-10-08T15:32:10
```

The timestamp is generated from the DS3231.

---

# 30. Storage Architecture

```text
Storage Manager
      │
      ├── SD Manager
      │
      ├── CSV Logger
      │
      └── System Logger
```

The Storage Manager handles SD initialization and availability.

The CSV Logger handles measurement records.

---

# 31. CSV Measurement Logging

Format:

```csv
timestamp,conductivity,tof_us,distance_mm,status
2026-10-08T15:32:10,4820,12.482,18.73,VALID
```

The logger should:

1. Create the directory if required.
2. Create the daily CSV file.
3. Add the header if the file is new.
4. Append the measurement.
5. Flush the file.
6. Close the file when appropriate.

The exact buffering strategy can be optimized after initial testing.

---

# 32. Storage Failure Handling

If the SD card is unavailable:

```text
Measurement
    │
    ▼
Valid Result
    │
    ├────► Display
    │
    ├────► Web API
    │
    └────► SD Logger
                │
                ▼
             FAILURE
                │
                ▼
          Warning Status
```

The measurement must not fail solely because the SD card is unavailable.

---

# 33. Configuration Manager

Configuration is stored using ESP-IDF NVS.

The Configuration Manager handles:

- Loading configuration.
- Saving configuration.
- Default values.
- Validation.
- Configuration versioning.

Example:

```cpp
struct AppConfig
{
    uint8_t ecSlaveId;

    float minConductivityUsCm;
    float maxConductivityUsCm;

    float minDistanceMm;
    float maxDistanceMm;

    float speedOfSoundMps;

    uint32_t measurementTimeoutMs;
};
```

---

# 34. Configuration Persistence

Configuration must survive reboot.

Startup behavior:

```text
Boot
 │
 ▼
Read NVS
 │
 ├── Valid configuration
 │       │
 │       ▼
 │    Load config
 │
 └── Invalid/missing
         │
         ▼
     Load defaults
         │
         ▼
     Save defaults
```

---

# 35. Configuration Versioning

NVS configuration should include a version number.

Example:

```cpp
struct ConfigMetadata
{
    uint16_t version;
};
```

If the firmware detects an incompatible version, it should migrate or reset the configuration to defaults.

---

# 36. Calibration Manager

The Calibration Manager handles:

- EC calibration configuration.
- TOF calibration.
- Calibration validation.
- Calibration persistence.

Calibration values must be stored in NVS.

---

# 37. EC Calibration

The SEN0707 follows the manufacturer's calibration procedure.

The supplied conductivity calibration solution can be used for verification.

The software should provide a calibration workflow without embedding sensor-specific assumptions into the UI.

Example:

```text
Calibration
     │
     ▼
Select EC Calibration
     │
     ▼
Prepare Calibration Solution
     │
     ▼
Read Sensor
     │
     ▼
Verify Value
     │
     ▼
Apply Calibration
     │
     ▼
Save Configuration
```

The exact calibration commands must follow the SEN0707 documentation.

---

# 38. TOF Calibration

V0.1 TOF calibration uses a known reference distance.

Example:

```text
Known Distance
      │
      ▼
Measure TOF
      │
      ▼
Calculate Error
      │
      ▼
Apply Calibration Factor/Offset
      │
      ▼
Save Calibration
```

The calibration model must remain replaceable.

The V0.1 air calibration must not be assumed valid for a future liquid/acoustic system.

---

# 39. Power Manager

The Power Manager handles simple battery monitoring.

Responsibilities:

- Read battery ADC.
- Convert ADC reading to battery voltage.
- Estimate battery percentage.
- Determine battery state.
- Notify UI.
- Report low-battery condition.

It does not measure current.

---

# 40. Battery Voltage Calculation

The battery voltage is measured through a resistor divider.

Conceptually:

```text
Battery Voltage
      │
      ▼
Voltage Divider
      │
      ▼
ESP32 ADC
      │
      ▼
ADC Voltage
      │
      ▼
Divider Calculation
      │
      ▼
Battery Voltage
```

The divider ratio must be stored in configuration or constants.

ADC calibration should be applied where appropriate.

---

# 41. Battery Percentage

A simple voltage-based estimate is used.

Example:

```cpp
uint8_t estimateBatteryPercent(float voltage);
```

The mapping is:

| Voltage | Estimated Level |
|---:|---:|
| ≥ 4.10 V | 100% |
| 4.00–4.09 V | 80% |
| 3.90–3.99 V | 60% |
| 3.80–3.89 V | 50% |
| 3.70–3.79 V | 35% |
| 3.60–3.69 V | 20% |
| 3.50–3.59 V | 10% |
| ≤ 3.40 V | Critical |

This is an approximate user-facing indicator.

It must not be presented as a precision state-of-charge measurement.

---

# 42. Low Battery Handling

Battery states:

```text
NORMAL
LOW
CRITICAL
```

Recommended behavior:

### NORMAL

Normal operation.

### LOW

- Display warning.
- Allow normal operation.
- Continue monitoring voltage.

### CRITICAL

- Display critical battery warning.
- Prevent new measurements if required.
- Save pending data where possible.
- Prepare for safe shutdown.

---

# 43. Wi-Fi Architecture

The ESP32-S3 operates as a local Wi-Fi access point.

Example:

```text
SSID:
EC-TOF-Analyzer

IP:
192.168.4.1
```

The exact SSID and network configuration should be configurable.

No internet connection is required.

---

# 44. Web Server

The web server provides a local browser interface.

```text
Browser
   │
   │ Wi-Fi
   ▼
ESP32-S3
   │
   ▼
HTTP Server
   │
   ├── Web UI
   │
   └── REST API
```

The web UI must consume application data.

It must not directly communicate with sensor drivers.

---

# 45. Web Pages

V0.1 should provide:

## Dashboard

Shows:

- Current conductivity.
- Current TOF.
- Distance.
- Battery.
- System status.
- Last measurement.

## Measurements

Shows:

- Measurement history.
- Measurement details.
- Timestamp.

## Calibration

Provides:

- EC calibration.
- TOF calibration.

## Configuration

Provides:

- Measurement settings.
- Sensor settings.
- Network settings.

## System Status

Shows:

- Firmware.
- Sensors.
- RTC.
- SD.
- Wi-Fi.
- Battery.

---

# 46. REST API

V0.1 API:

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

---

# 47. Device API

Example:

```json
{
    "device": "EC-TOF-Analyzer",
    "firmware": "0.1.0",
    "hardware": "V0.1"
}
```

---

# 48. System Status API

Example:

```json
{
    "status": "READY",
    "battery_voltage": 3.82,
    "battery_percent": 52,
    "battery_state": "NORMAL",
    "rtc": "OK",
    "sd": "OK",
    "ec_sensor": "OK",
    "tof_sensor": "OK",
    "wifi": "AP"
}
```

There are intentionally no battery current or battery power fields.

---

# 49. Current Measurement API

Example:

```json
{
    "timestamp": "2026-10-08T15:32:10",
    "conductivity_us_cm": 4820,
    "tof_us": 12.482,
    "distance_mm": 18.73,
    "status": "VALID"
}
```

---

# 50. Start Measurement API

Request:

```http
POST /api/measurement/start
```

Example response:

```json
{
    "accepted": true,
    "status": "MEASURING"
}
```

The API should not block while waiting for the measurement to finish.

---

# 51. Measurement History API

```http
GET /api/measurements
```

The API may return recent measurements.

Example:

```json
{
    "count": 2,
    "measurements": [
        {
            "timestamp": "2026-10-08T15:32:10",
            "conductivity_us_cm": 4820,
            "tof_us": 12.482,
            "distance_mm": 18.73,
            "status": "VALID"
        }
    ]
}
```

---

# 52. Firmware Update

V0.1 firmware updates are performed through USB.

Web-based OTA is not required for the initial prototype.

A future version may add:

```text
Web UI
   │
   ▼
Firmware Upload
   │
   ▼
OTA
   │
   ▼
ESP32-S3
```

---

# 53. FreeRTOS Task Architecture

The firmware should use a limited number of tasks.

Recommended tasks:

```text
Main/Application Task
        │
        ├── Measurement Task
        ├── UI Task
        ├── Storage Task
        └── Web Task
```

Not every module requires its own task.

Drivers should normally execute within the task that owns the operation.

---

# 54. Measurement Task

Responsibilities:

- Process measurement requests.
- Run the measurement state machine.
- Communicate with TOF.
- Communicate with EC.
- Validate results.
- Publish measurement results.

The task should not directly render the UI.

---

# 55. UI Task

Responsibilities:

- Run LVGL.
- Process display updates.
- Process user interface events.
- Render status.

The UI task receives application state.

---

# 56. Storage Task

The storage task may be used to prevent SD writes from delaying measurement operations.

Example:

```text
Measurement Task
      │
      ▼
Measurement Queue
      │
      ▼
Storage Task
      │
      ▼
MicroSD
```

If initial testing shows SD operations are fast enough, the architecture may be simplified.

---

# 57. Web Task

The ESP-IDF HTTP server handles web requests.

API handlers should communicate with application managers.

They should not directly access low-level drivers.

---

# 58. Inter-Task Communication

Possible mechanisms:

- FreeRTOS queues.
- Event groups.
- Mutexes.
- Notifications.

Use the simplest mechanism that meets the requirement.

### Example

```text
START Button
     │
     ▼
Measurement Request Queue
     │
     ▼
Measurement Task
```

---

# 59. Shared State

Shared application state should be protected.

Examples:

- Current measurement.
- System status.
- Battery status.
- Sensor status.
- Configuration.

Use a mutex or controlled ownership model where required.

Avoid unrestricted global variables.

---

# 60. Event-Based System Status

Important system events should be represented consistently.

Example:

```cpp
enum class SystemEvent
{
    SYSTEM_READY,
    MEASUREMENT_STARTED,
    MEASUREMENT_COMPLETE,
    MEASUREMENT_FAILED,
    EC_SENSOR_ERROR,
    TOF_SENSOR_ERROR,
    SD_ERROR,
    RTC_ERROR,
    LOW_BATTERY
};
```

---

# 61. Error Handling

Errors should be categorized.

```text
INFO
WARNING
ERROR
CRITICAL
```

Examples:

### WARNING

- Low battery.
- SD card unavailable.

### ERROR

- EC sensor timeout.
- TOF timeout.
- RTC communication failure.

### CRITICAL

- Critical battery.
- Invalid power condition.
- Hardware condition requiring shutdown.

---

# 62. Sensor Recovery

The firmware should attempt recovery where practical.

Example:

```text
EC Timeout
    │
    ▼
Retry
    │
    ├── Success → Continue
    │
    └── Failure
          │
          ▼
     Sensor Error
```

The retry count must be limited.

The firmware must never retry indefinitely.

---

# 63. Watchdog Strategy

The ESP32-S3 watchdog should be enabled according to ESP-IDF recommendations.

Long operations must yield appropriately.

Potential watchdog-sensitive areas:

- SD operations.
- Wi-Fi operations.
- LVGL processing.
- Sensor communication.

No blocking loop should run indefinitely.

---

# 64. Logging

Firmware logging should use ESP-IDF logging.

Example:

```cpp
ESP_LOGI(TAG, "Measurement started");
ESP_LOGI(TAG, "Conductivity: %.2f uS/cm", conductivity);
ESP_LOGI(TAG, "TOF: %.3f us", tof);
ESP_LOGE(TAG, "SEN0707 timeout");
```

Use appropriate log levels:

```text
ESP_LOGE
ESP_LOGW
ESP_LOGI
ESP_LOGD
ESP_LOGV
```

Production builds may reduce debug logging.

---

# 65. Diagnostics

The System Manager should provide a diagnostics summary.

Example:

```cpp
struct DiagnosticStatus
{
    bool rtcOk;
    bool sdOk;
    bool ecOk;
    bool tofOk;
    bool displayOk;
    bool wifiOk;
};
```

The diagnostics screen should present these statuses to the user.

---

# 66. Measurement Data Flow

```text
START
 │
 ▼
Measurement Manager
 │
 ├───────────────► TOF Manager
 │                    │
 │                    ▼
 │                 TOF Data
 │                    │
 │                    ▼
 │                Distance
 │
 └───────────────► EC Manager
                      │
                      ▼
                  EC Data
                      │
                      ▼
              Measurement Data
                      │
              ┌───────┼────────┐
              ▼       ▼        ▼
           Display   SD       Web
```

---

# 67. Measurement Result Ownership

The Measurement Manager owns the completed measurement result.

Other modules receive a copy or immutable view.

This prevents:

- Display modifying measurement data.
- Web API modifying measurement data.
- Storage modifying measurement data.

---

# 68. Thread Safety

Shared objects must have defined ownership.

Examples:

- Measurement data: Measurement Manager owns it.
- UI state: UI Manager owns it.
- Configuration: Configuration Manager owns it.
- Battery state: Power Manager owns it.
- Storage state: Storage Manager owns it.

Cross-module access should use interfaces.

---

# 69. Timing Requirements

The firmware must support microsecond-level timing for HC-SR04 echo measurement.

Use appropriate ESP-IDF timing facilities.

Do not use:

```cpp
vTaskDelay()
```

for the actual echo pulse measurement.

Task delays may be used for normal application scheduling.

---

# 70. Configuration Defaults

Initial defaults should include:

```text
EC Slave ID:
1

Measurement Timeout:
Configurable

Minimum Conductivity:
10 µS/cm

Maximum Conductivity:
20,000 µS/cm

Speed of Sound:
Configurable

Minimum Distance:
Configurable

Maximum Distance:
Configurable

Wi-Fi SSID:
EC-TOF-Analyzer
```

Actual limits should be validated against the customer's sample and measurement requirements.

---

# 71. Security Considerations

V0.1 operates as a local device.

Basic protections should include:

- Do not expose the device to the public internet.
- Validate web request parameters.
- Validate configuration values.
- Limit firmware upload functionality to USB.
- Avoid unsafe memory operations.
- Validate file paths.
- Restrict file access to the application storage directory.

Authentication may be added in a later version if required.

---

# 72. Resource Management

The ESP32-S3 has limited embedded resources.

The firmware should:

- Avoid unnecessary dynamic allocation.
- Avoid large temporary buffers.
- Reuse buffers where practical.
- Limit JSON response sizes.
- Limit web history queries.
- Avoid loading entire CSV files into RAM.
- Stream large files when required.
- Monitor heap usage during development.

---

# 73. SD File Handling

The software should avoid keeping files open indefinitely.

Recommended pattern:

```text
Open
  │
Write
  │
Flush
  │
Close
```

For high-frequency logging, buffering may be introduced after performance testing.

V0.1 measurements are expected to be low frequency, so reliability is more important than maximum write performance.

---

# 74. Web Data Handling

The web interface should display the same application data used by the TFT.

There should be one source of truth.

```text
                 Application State
                       │
              ┌────────┴────────┐
              ▼                 ▼
          TFT UI             Web UI
```

The web UI must not maintain an independent measurement state.

---

# 75. Calibration Data Storage

Calibration data should be stored in NVS.

Example:

```cpp
struct CalibrationData
{
    uint16_t version;

    float ecOffset;
    float ecScale;

    float tofOffset;
    float tofScale;
};
```

The actual EC calibration parameters must follow the SEN0707 capabilities.

---

# 76. Factory Reset

The configuration system should support factory reset.

Factory reset should restore:

- Default configuration.
- Default calibration state where appropriate.
- Default Wi-Fi settings.

Measurement history on the SD card should not be deleted automatically unless explicitly requested.

---

# 77. Firmware Version

Firmware versions should follow semantic versioning where practical.

Example:

```text
0.1.0
```

The device should expose the firmware version through:

- TFT system screen.
- Web UI.
- `/api/device`.

---

# 78. Development Build

Development builds should enable:

- Detailed logging.
- Diagnostics.
- Sensor communication logging.
- Heap monitoring.
- Error reporting.

Production-like builds can reduce diagnostic output after the prototype is stable.

---

# 79. Unit Testing

Unit-testable components should be isolated from ESP32 hardware where practical.

Candidates:

- Distance calculation.
- Battery percentage calculation.
- Measurement validation.
- Configuration validation.
- CSV formatting.
- Calibration calculations.
- Modbus CRC calculation.

Example:

```cpp
float calculateDistanceMm(
    float tofUs,
    float speedOfSoundMps
);
```

This function can be tested without hardware.

---

# 80. Integration Testing

Hardware integration testing should cover:

### EC

- RS485 communication.
- Modbus request.
- Modbus response.
- CRC.
- Conductivity reading.
- Sensor disconnect.

### TOF

- Trigger.
- Echo.
- Timeout.
- Distance calculation.

### Storage

- SD initialization.
- File creation.
- CSV logging.
- SD removal.

### RTC

- Read time.
- Set time.
- Timestamp generation.

### Battery

- ADC reading.
- Voltage calculation.
- Percentage estimation.
- Low-battery state.

---

# 81. End-to-End Test

The complete measurement test is:

```text
Power ON
   │
   ▼
System READY
   │
   ▼
Press START
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
   ├── Invalid → Error
   │
   ▼
Display Result
   │
   ▼
Timestamp Result
   │
   ▼
Write CSV
   │
   ▼
Update Web Data
   │
   ▼
READY
```

---

# 82. Failure Test

The firmware should be tested with:

- EC sensor disconnected.
- RS485 cable disconnected.
- HC-SR04 disconnected.
- HC-SR04 no echo.
- SD card removed.
- RTC disconnected.
- Low battery.
- Critical battery.
- Invalid configuration.
- Wi-Fi client disconnect.
- Repeated measurement requests.

The system must remain recoverable.

---

# 83. Software Boundaries

The following rules are mandatory.

### UI must not:

- Read UART directly.
- Read RS485 directly.
- Read GPIO sensors directly.
- Write SD files directly.

### Web API must not:

- Directly access sensors.
- Directly manipulate GPIO.
- Directly write NVS.

### Sensor drivers must not:

- Update UI.
- Write SD files.
- Handle web requests.

### Storage must not:

- Control sensors.
- Start measurements.

### Measurement Manager must:

- Coordinate the measurement process.
- Own the measurement state.

---

# 84. Dependency Direction

Dependencies should flow downward:

```text
UI
 │
 ▼
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

Lower layers must not depend on higher layers.

For example:

```text
HC-SR04 Driver
```

must not include:

```text
DisplayManager
```

---

# 85. Replaceable TOF Architecture

The most important future-proofing requirement is the TOF interface.

Current implementation:

```text
ITofSensor
    │
    └── HcSr04
```

Future implementation:

```text
ITofSensor
    │
    ├── HcSr04
    │
    ├── LaboratoryUltrasonic
    │
    └── CustomTofReceiver
```

The Measurement Manager should not require modification when the underlying TOF hardware changes, except where the measurement model itself changes.

---

# 86. Replaceable EC Architecture

The EC subsystem follows the same concept.

```text
IEcSensor
    │
    └── Sen0707
```

Future:

```text
IEcSensor
    │
    ├── Sen0707
    └── OtherEcSensor
```

This allows a different EC sensor to be tested without rewriting the measurement application.

---

# 87. Future Liquid TOF Support

The V0.1 firmware must clearly separate:

1. Raw TOF acquisition.
2. Distance calculation.
3. Measurement model.

The HC-SR04 air formula is:

```text
Distance = TOF × Speed of Sound / 2
```

A future liquid/acoustic implementation may require:

- Different sound velocity.
- Different acoustic path.
- Transducer delay compensation.
- Signal detection.
- Gain control.
- Cross-correlation.
- Temperature compensation.
- Sample cell geometry.

These must not be hardcoded into the V0.1 HC-SR04 driver.

---

# 88. Application State

The application should maintain a central runtime state.

Example:

```cpp
struct ApplicationState
{
    SystemStatus systemStatus;

    MeasurementStatus measurementStatus;

    MeasurementData currentMeasurement;

    PowerStatus powerStatus;

    DiagnosticStatus diagnostics;
};
```

This state feeds:

- TFT UI.
- Web API.
- System diagnostics.

---

# 89. Measurement Request

A measurement request can be represented as:

```cpp
struct MeasurementRequest
{
    uint32_t requestId;
    bool requested;
};
```

The request is placed into the measurement system rather than directly calling the measurement function from the UI.

---

# 90. Measurement Result

```cpp
struct MeasurementResult
{
    bool success;

    MeasurementData data;

    MeasurementStatus status;

    ErrorCode error;
};
```

An error code system should be defined centrally.

---

# 91. Error Codes

Example:

```cpp
enum class ErrorCode
{
    NONE,

    EC_TIMEOUT,
    EC_CRC_ERROR,
    EC_INVALID_RESPONSE,
    EC_DISCONNECTED,

    TOF_TIMEOUT,
    TOF_INVALID,
    TOF_DISCONNECTED,

    RTC_ERROR,

    SD_NOT_FOUND,
    SD_WRITE_ERROR,

    BATTERY_LOW,
    BATTERY_CRITICAL,

    CONFIG_INVALID
};
```

The final list can expand during implementation.

---

# 92. Logging and Measurement Separation

Two different logging systems should be maintained.

### Measurement log

CSV data intended for users and analysis.

### System log

Diagnostic information intended for development and troubleshooting.

They should not be mixed.

---

# 93. Memory Management

Avoid unnecessary heap allocation during measurements.

Preferred approach:

```text
Initialize
   │
   ▼
Allocate required buffers
   │
   ▼
Reuse buffers
   │
   ▼
Run measurements
```

Do not repeatedly allocate and free memory inside high-frequency measurement loops.

---

# 94. Watchdog Recovery

If a recoverable task fails:

```text
Task Failure
    │
    ▼
Error Log
    │
    ▼
Attempt Recovery
    │
    ├── Success → Continue
    │
    └── Failure → System Error
```

A full device reboot should only be used when required.

---

# 95. Startup Diagnostics

At startup, the firmware should check:

```text
ESP32-S3             OK
Configuration        OK
RTC                  OK
SD                   OK
Display              OK
EC Interface         OK
TOF Interface        OK
Battery ADC          OK
Wi-Fi                OK
```

The result should be available through the System Status screen.

---

# 96. Software Acceptance Criteria

The software is ready for V0.1 validation when:

- ESP32-S3 boots reliably.
- Configuration loads correctly.
- SEN0707 can be read.
- RS485 errors are detected.
- HC-SR04 TOF can be measured.
- Echo timeout is handled.
- Distance is calculated.
- Measurements are validated.
- Results appear on the TFT.
- Physical controls operate correctly.
- RTC timestamps are correct.
- Measurements are logged to SD.
- SD failures do not crash the application.
- Battery voltage is displayed.
- Battery percentage is displayed.
- Low battery is detected.
- Wi-Fi AP starts correctly.
- Web dashboard displays measurement data.
- REST API works.
- Configuration can be changed.
- Calibration data persists.
- Sensor failures are reported.
- Measurement sequence returns to READY after completion or failure.

---

# 97. V0.1 Software Exclusions

The following are excluded from V0.1:

- Battery current measurement.
- Battery power measurement.
- INA226 support.
- Dedicated fuel gauge.
- Cloud backend.
- Remote internet access.
- Mobile application.
- Web OTA.
- Production-grade ultrasonic signal processing.
- Advanced acoustic signal analysis.
- Automatic sample identification.
- Advanced analytics.
- Multi-device synchronization.

---

# 98. Final Software Architecture

```text
                         ┌─────────────────────────┐
                         │       TFT / LVGL         │
                         │       Web Browser        │
                         └────────────┬────────────┘
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │    Application Layer    │
                         │                         │
                         │ Measurement Manager     │
                         │ Configuration Manager   │
                         │ Calibration Manager     │
                         │ System Manager          │
                         │ Power Manager           │
                         └────────────┬────────────┘
                                      │
                 ┌────────────────────┼────────────────────┐
                 │                    │                    │
                 ▼                    ▼                    ▼
          ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
          │ EC Manager  │      │ TOF Manager │      │  Storage    │
          │             │      │             │      │  Manager    │
          └──────┬──────┘      └──────┬──────┘      └──────┬──────┘
                 │                    │                    │
                 ▼                    ▼                    ▼
          ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
          │ SEN0707     │      │ HC-SR04     │      │ MicroSD     │
          │ Modbus      │      │ Driver      │      │ CSV         │
          └──────┬──────┘      └─────────────┘      └─────────────┘
                 │
                 ▼
             MAX3485
                 │
                 ▼
              RS485

                 ┌──────────────────────────┐
                 │        Services          │
                 │                          │
                 │ RTC │ Wi-Fi │ Web API    │
                 └──────────────────────────┘

                 ┌──────────────────────────┐
                 │       Hardware Drivers   │
                 │                          │
                 │ SPI │ I2C │ GPIO │ ADC   │
                 │ UART │ RS485 │ TFT       │
                 └──────────────────────────┘
```

---

# 99. Final V0.1 Software Boundary

The firmware architecture is intentionally designed around the following principle:

```text
                    APPLICATION
                         │
          ┌──────────────┴──────────────┐
          │                             │
       EC Sensor                    TOF Sensor
          │                             │
      SEN0707                       HC-SR04
          │                             │
       RS485                         GPIO
```

The application knows what measurement it needs.

It does not need to know the low-level implementation details of the sensor.

This allows the V0.1 prototype to validate the complete EC-TOF workflow while keeping the software architecture ready for a future laboratory-grade ultrasonic measurement system.