# EC-TOF Analyzer

## Software Design

Version: 0.1  
Status: Prototype Planning  
Related Documents:

- `01-product-requirements.md`
- `02-technical-requirements.md`
- `03-system-architecture.md`
- `04-hardware-design.md`

---

# 1. Purpose

This document defines the software architecture and implementation strategy for the EC-TOF Analyzer V0.1 firmware.

The firmware will run on the ESP32-S3 using ESP-IDF.

The software is designed to:

- Read conductivity from the SEN0707.
- Communicate with the SEN0707 through RS485 Modbus RTU.
- Trigger and measure the HC-SR04 ultrasonic sensor.
- Calculate ultrasonic time of flight.
- Calculate distance from the measured TOF.
- Display measurements on the TFT.
- Accept input from physical controls.
- Timestamp measurements using the DS3231.
- Store measurements on MicroSD.
- Provide a local Wi-Fi web interface.
- Manage calibration.
- Manage configuration.
- Detect and report hardware errors.
- Keep sensor-specific implementations modular.

---

# 2. Software Design Principles

The firmware shall follow these principles:

1. Use ESP-IDF as the primary framework.
2. Use C++ for application and driver code where practical.
3. Keep `main.cpp` minimal.
4. Separate hardware drivers from application logic.
5. Use interfaces for replaceable sensors.
6. Keep measurement logic independent from the display.
7. Keep measurement logic independent from the web interface.
8. Use FreeRTOS for concurrent system activities where required.
9. Avoid blocking operations inside time-critical measurement code.
10. Use deterministic state machines for measurement operations.
11. Treat SD storage as asynchronous where practical.
12. Validate sensor data before logging.
13. Store configuration separately from measurement data.
14. Keep V0.1 implementation simple enough to debug on hardware.

---

# 3. Firmware Architecture

The firmware is divided into five primary layers:

```text
┌──────────────────────────────────────────┐
│              User Interfaces             │
│                                          │
│          TFT UI        Web UI            │
└───────────────────┬──────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────┐
│           Application Services           │
│                                          │
│ Measurement │ Calibration │ Configuration│
│ System      │ History     │ Power        │
└───────────────────┬──────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────┐
│             Device Services              │
│                                          │
│ EC │ TOF │ RTC │ Storage │ Input │ Wi-Fi │
└───────────────────┬──────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────┐
│             Hardware Drivers             │
│                                          │
│ UART │ RS485 │ SPI │ I2C │ GPIO │ Timer  │
└───────────────────┬──────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────┐
│                 Hardware                 │
└──────────────────────────────────────────┘
```

---

# 4. ESP-IDF Project Structure

Recommended project structure:

```text
ec-tof-analyzer/
│
├── CMakeLists.txt
├── sdkconfig
├── sdkconfig.defaults
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
│   │   └── configuration_manager.h
│   │
│   ├── sensors/
│   │   ├── ec/
│   │   │   ├── ec_sensor.h
│   │   │   ├── sen0707.cpp
│   │   │   └── sen0707.h
│   │   │
│   │   └── tof/
│   │       ├── tof_sensor.h
│   │       ├── hcsr04.cpp
│   │       └── hcsr04.h
│   │
│   ├── drivers/
│   │   ├── rs485/
│   │   ├── uart/
│   │   ├── spi/
│   │   ├── i2c/
│   │   ├── gpio/
│   │   └── timer/
│   │
│   ├── display/
│   │   ├── display_manager.cpp
│   │   ├── display_manager.h
│   │   ├── screens/
│   │   └── widgets/
│   │
│   ├── input/
│   │   ├── input_manager.cpp
│   │   ├── input_manager.h
│   │   └── encoder.cpp
│   │
│   ├── storage/
│   │   ├── storage_manager.cpp
│   │   ├── storage_manager.h
│   │   └── csv_logger.cpp
│   │
│   ├── rtc/
│   │   ├── rtc_manager.cpp
│   │   └── rtc_manager.h
│   │
│   ├── web/
│   │   ├── web_server.cpp
│   │   ├── web_server.h
│   │   ├── api/
│   │   └── www/
│   │
│   ├── power/
│   │   ├── power_manager.cpp
│   │   └── power_manager.h
│   │
│   ├── system/
│   │   ├── system_manager.cpp
│   │   ├── system_manager.h
│   │   └── system_events.h
│   │
│   └── common/
│       ├── types.h
│       ├── constants.h
│       ├── errors.h
│       └── utilities.h
│
└── components/
```

The exact directory structure can be simplified during implementation if a module does not require its own abstraction.

---

# 5. Main Entry Point

`main.cpp` should remain minimal.

Conceptual implementation:

```cpp
extern "C" void app_main()
{
    AppManager app;

    app.initialize();
    app.start();
}
```

Application initialization should be handled by dedicated managers.

---

# 6. Common Data Types

Shared types shall be defined in:

```text
main/common/types.h
```

Example measurement structure:

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

Example status:

```cpp
enum class MeasurementStatus
{
    INVALID,
    VALID,
    EC_ERROR,
    TOF_ERROR,
    TIMEOUT,
    STORAGE_ERROR
};
```

---

# 7. Measurement Manager

The Measurement Manager is the core application service.

Responsibilities:

- Start measurement.
- Coordinate EC and TOF sensors.
- Validate results.
- Calculate derived values.
- Generate measurement records.
- Publish measurement events.
- Send completed records to storage.
- Update UI state.

The Measurement Manager shall not directly control TFT rendering or write files.

---

# 8. Measurement Manager Interface

Conceptual interface:

```cpp
class MeasurementManager
{
public:

    bool initialize();

    bool startMeasurement();

    bool isBusy() const;

    Measurement getCurrentMeasurement();

private:

    void runMeasurement();

    bool measureTof();

    bool measureEc();

    bool validateMeasurement();
};
```

The exact interface may change during implementation.

---

# 9. Measurement State Machine

The measurement process shall use an explicit state machine.

```text
                    ┌─────────────┐
                    │    IDLE     │
                    └──────┬──────┘
                           │ START
                           ▼
                  ┌──────────────────┐
                  │ MEASUREMENT_START│
                  └────────┬─────────┘
                           ▼
                    ┌────────────┐
                    │ TOF_TRIGGER│
                    └─────┬──────┘
                          ▼
                    ┌────────────┐
                    │  TOF_WAIT  │
                    └─────┬──────┘
                          │
                 ┌────────┴────────┐
                 │                 │
              Success            Timeout
                 │                 │
                 ▼                 ▼
           TOF_PROCESS           ERROR
                 │
                 ▼
               EC_READ
                 │
                 ▼
              VALIDATE
                 │
          ┌──────┴──────┐
          │             │
        Valid         Invalid
          │             │
          ▼             ▼
       DISPLAY         ERROR
          │
          ▼
          LOG
          │
          ▼
         READY
```

---

# 10. Ultrasonic Driver

The ultrasonic driver shall abstract the HC-SR04.

Interface:

```cpp
class ITofSensor
{
public:

    virtual bool initialize() = 0;

    virtual bool trigger() = 0;

    virtual bool waitForEcho(uint32_t timeout_us) = 0;

    virtual float getTofUs() = 0;

    virtual float getDistanceMm() = 0;

    virtual bool isConnected() = 0;
};
```

The HC-SR04 implementation shall implement this interface.

---

# 11. HC-SR04 Measurement

The HC-SR04 sequence is:

```text
Set TRIG LOW
      ↓
Wait
      ↓
Set TRIG HIGH
      ↓
Generate trigger pulse
      ↓
Set TRIG LOW
      ↓
Wait for ECHO HIGH
      ↓
Start timer
      ↓
Wait for ECHO LOW
      ↓
Stop timer
      ↓
Calculate TOF
```

The implementation shall use a hardware timing mechanism suitable for microsecond measurement.

Possible ESP32-S3 implementation options include:

- RMT
- GPTimer
- GPIO interrupt + hardware timer

The selected implementation should prioritize reliable timing and avoid long blocking delays.

---

# 12. TOF Calculation

The driver shall provide the measured echo duration.

Example:

```text
TOF = Echo High Duration
```

Distance calculation will depend on the validated measurement geometry.

For an air ultrasonic measurement:

```text
distance = TOF × speed_of_sound / 2
```

The speed-of-sound assumption must not be blindly reused for a future liquid/acoustic implementation.

The V0.1 firmware shall keep the distance calculation configurable so that the final ultrasonic hardware and propagation medium can be changed later.

---

# 13. EC Sensor Interface

The EC sensor shall use an abstraction.

```cpp
class IEcSensor
{
public:

    virtual bool initialize() = 0;

    virtual bool readConductivity(float& value) = 0;

    virtual bool isConnected() = 0;

    virtual bool calibrate() = 0;
};
```

The SEN0707 implementation shall use Modbus RTU.

---

# 14. SEN0707 Driver

The driver shall handle:

- Modbus requests.
- Modbus responses.
- CRC validation.
- Register parsing.
- Timeout handling.
- Retry handling.
- Conductivity conversion.
- Sensor communication errors.

The driver shall not:

- Draw UI elements.
- Write SD files.
- Manage Wi-Fi.
- Control measurement states.

---

# 15. Modbus RTU Architecture

```text
EC Service
    ↓
SEN0707 Driver
    ↓
Modbus RTU
    ↓
RS485 Driver
    ↓
UART Driver
    ↓
MAX3485
    ↓
SEN0707
```

The Modbus layer should provide generic operations where practical.

Example:

```cpp
bool readHoldingRegisters(
    uint8_t address,
    uint16_t registerAddress,
    uint16_t count,
    uint16_t* data
);
```

Sensor-specific register mapping belongs in the SEN0707 driver.

---

# 16. RS485 Driver

The RS485 driver shall manage:

- UART initialization.
- TX/RX.
- Direction control.
- Communication timeout.
- Buffer handling.

Conceptual interface:

```cpp
class Rs485Driver
{
public:

    bool initialize();

    bool transmit(
        const uint8_t* data,
        size_t length
    );

    int receive(
        uint8_t* buffer,
        size_t length,
        uint32_t timeout_ms
    );
};
```

---

# 17. Modbus Error Handling

Possible errors:

```text
MODBUS_TIMEOUT
MODBUS_CRC_ERROR
MODBUS_EXCEPTION
MODBUS_INVALID_RESPONSE
MODBUS_UART_ERROR
RS485_ERROR
SENSOR_DISCONNECTED
```

The sensor driver shall convert low-level errors into application-level sensor status.

---

# 18. Display Architecture

The display system shall be separated into:

```text
Display Manager
      ↓
Screen Manager
      ↓
UI Components
      ↓
ST7796 Driver
      ↓
SPI
```

LVGL may be used for the UI layer if the selected display driver and project configuration support it cleanly.

The display driver shall remain independent from measurement logic.

---

# 19. Main Display

The main screen should show:

```text
┌─────────────────────────────────────┐
│          EC-TOF ANALYZER            │
│                                     │
│ Conductivity                        │
│ 4.82 mS/cm                          │
│                                     │
│ TOF                                 │
│ 12.482 µs                           │
│                                     │
│ Distance                            │
│ 18.73 mm                            │
│                                     │
│ Status: READY                       │
│                                     │
│        [ START ]                    │
└─────────────────────────────────────┘
```

The exact visual design will be defined separately.

---

# 20. UI Screens

V0.1 screens:

```text
1. Main Measurement
2. Measurement History
3. Calibration
4. Configuration
5. System Status
6. About
```

Navigation shall be optimized for the rotary encoder.

---

# 21. UI Navigation

Example:

```text
MAIN
 │
 ├── Start Measurement
 │
 ├── History
 │
 ├── Calibration
 │    ├── EC Calibration
 │    └── TOF Calibration
 │
 ├── Configuration
 │
 └── System Status
```

START should provide a direct measurement action without requiring menu navigation.

---

# 22. Input Manager

The Input Manager converts physical inputs into application events.

Example:

```cpp
enum class InputEvent
{
    NONE,
    ENCODER_UP,
    ENCODER_DOWN,
    SELECT,
    START,
    BACK
};
```

The UI should consume these events rather than reading GPIO pins directly.

---

# 23. Input Debouncing

Mechanical buttons shall use software debouncing.

The debounce period should be configurable.

The encoder should also use suitable filtering to prevent false transitions.

---

# 24. RTC Manager

The RTC Manager abstracts the DS3231.

Interface:

```cpp
class RtcManager
{
public:

    bool initialize();

    bool getDateTime(DateTime& time);

    bool setDateTime(const DateTime& time);

    bool isAvailable();
};
```

The RTC is the source of timestamps for measurement records.

---

# 25. Time Synchronization

V0.1 does not require internet time.

The system should support:

```text
RTC time
```

Optionally, the web interface may provide a manual date/time setting function.

Future versions may support NTP when connected to a network.

---

# 26. Storage Manager

The Storage Manager abstracts the MicroSD card.

Responsibilities:

- Mount filesystem.
- Create directories.
- Create measurement files.
- Write CSV records.
- Flush data.
- Read stored measurements.
- Export files.
- Detect SD errors.

Interface:

```cpp
class StorageManager
{
public:

    bool initialize();

    bool isAvailable();

    bool writeMeasurement(
        const Measurement& measurement
    );

    bool readHistory();

    bool exportData();
};
```

---

# 27. Storage Directory Structure

Recommended:

```text
/ECTOF/
│
├── config/
│
├── data/
│   └── YYYY/
│       └── MM/
│           └── DD.csv
│
└── logs/
```

Example:

```text
/ECTOF/data/2026/10/08.csv
```

---

# 28. CSV Format

Measurement records shall use:

```text
timestamp,conductivity,tof_us,distance_mm,status
```

Example:

```text
2026-10-08T15:32:10,4820,12.482,18.73,VALID
```

CSV shall remain the primary measurement export format for V0.1.

---

# 29. Logging Strategy

A measurement shall be written after the measurement is validated.

Sequence:

```text
Measurement Complete
        ↓
Create Record
        ↓
Queue Record
        ↓
Storage Task
        ↓
Write CSV
        ↓
Flush
```

The measurement process should not wait unnecessarily for slow SD operations.

---

# 30. SD Error Handling

Possible errors:

```text
SD_NOT_PRESENT
SD_MOUNT_FAILED
SD_WRITE_FAILED
SD_READ_FAILED
SD_FILE_ERROR
SD_FULL
```

A storage failure should not prevent the user from continuing measurements unless data integrity requires the measurement process to stop.

The UI shall clearly indicate when logging is unavailable.

---

# 31. Configuration Manager

Configuration values shall be centralized.

Possible configuration:

```cpp
struct DeviceConfig
{
    uint8_t modbusAddress;

    uint32_t modbusTimeoutMs;

    uint8_t measurementRetries;

    float tofCalibrationOffset;

    float tofScaleFactor;

    float ecCalibrationFactor;

    uint32_t displayBrightness;

    bool autoLogging;

    uint32_t measurementTimeoutMs;
};
```

Only required configuration values should be implemented in V0.1.

---

# 32. Configuration Storage

Configuration may be stored using ESP-IDF NVS.

Architecture:

```text
Configuration Manager
        ↓
ESP-IDF NVS
        ↓
Flash
```

Measurement records remain on MicroSD.

NVS should not be used for large measurement datasets.

---

# 33. Calibration Manager

The Calibration Manager shall control calibration workflows.

Architecture:

```text
Calibration UI
      ↓
Calibration Manager
      ├── EC Calibration
      └── TOF Calibration
              ↓
       Configuration
```

---

# 34. EC Calibration

The EC calibration process shall follow the selected SEN0707 calibration procedure.

The application should provide:

```text
Enter Calibration
      ↓
Prepare Standard
      ↓
Read Sensor
      ↓
Validate Reading
      ↓
Apply Calibration
      ↓
Save Configuration
      ↓
Calibration Complete
```

The calibration standard and exact procedure shall be based on the sensor manufacturer's documented method.

---

# 35. TOF Calibration

The V0.1 TOF calibration should use a known reference distance.

Example:

```text
Known Distance
      ↓
Perform Measurement
      ↓
Compare Measured Distance
      ↓
Calculate Correction
      ↓
Save Calibration
```

A simple correction model may initially use:

```text
corrected_distance =
    measured_distance × scale + offset
```

The model can be expanded later if required.

---

# 36. System Manager

The System Manager coordinates global system state.

Possible states:

```text
BOOTING
INITIALIZING
READY
MEASURING
ERROR
CALIBRATION
LOW_BATTERY
```

The System Manager shall provide a single source of truth for system status.

---

# 37. Power Manager

The Power Manager shall monitor:

- Battery voltage.
- Low-battery condition.
- Power rail status where available.

Future versions may control:

- Sensor power.
- Display backlight.
- Peripheral power.
- Sleep modes.

---

# 38. Wi-Fi Architecture

V0.1 shall operate as a local Wi-Fi access point.

Example:

```text
SSID:
EC-TOF-Analyzer
```

Example address:

```text
192.168.4.1
```

The exact SSID and password should be configurable.

---

# 39. Web Server

The ESP32-S3 shall host a local HTTP server.

Architecture:

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
```

The web server shall not directly communicate with sensors.

---

# 40. REST API

Initial API:

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

The API should remain small in V0.1.

---

# 41. Web Dashboard

The dashboard should display:

```text
EC-TOF ANALYZER

Conductivity
4.82 mS/cm

TOF
12.482 µs

Distance
18.73 mm

Status
READY

[ START MEASUREMENT ]
```

Additional information:

- SD status
- RTC status
- Sensor status
- Battery status

---

# 42. Web Measurement Flow

When the user selects START:

```text
Browser
   ↓
POST /api/measurement/start
   ↓
Measurement Manager
   ↓
Measurement Sequence
   ↓
Measurement Result
   ↓
Browser polls current result
```

The HTTP request should not remain blocked for the entire measurement duration if the measurement can take significant time.

---

# 43. Web History

The web UI shall allow the user to view stored measurements.

Example:

```text
Date        EC       TOF       Distance
------------------------------------------------
2026-10-08  4820     12.482    18.73
2026-10-08  4831     12.510    18.77
2026-10-08  4812     12.461    18.70
```

CSV export shall be available.

---

# 44. Web Calibration

Calibration controls should be protected from accidental activation.

The interface should clearly indicate:

```text
Calibration Mode
```

before modifying calibration parameters.

---

# 45. System Event Architecture

System events should use a common event mechanism.

Example:

```cpp
enum class SystemEvent
{
    SYSTEM_READY,
    MEASUREMENT_STARTED,
    MEASUREMENT_COMPLETE,
    MEASUREMENT_ERROR,
    SENSOR_ERROR,
    SD_ERROR,
    RTC_ERROR,
    LOW_BATTERY
};
```

Events may be distributed through FreeRTOS queues or ESP-IDF event mechanisms.

---

# 46. FreeRTOS Tasks

Initial task architecture:

```text
┌───────────────────────────────┐
│          FreeRTOS             │
│                               │
│ System Task                   │
│ Measurement Task              │
│ UI Task                       │
│ Storage Task                  │
│ Web Task                      │
│ Input Task                    │
└───────────────────────────────┘
```

The final implementation should avoid creating tasks without a clear need.

---

# 47. Measurement Task

Responsibilities:

```text
Receive measurement request
        ↓
Run state machine
        ↓
Read TOF
        ↓
Read EC
        ↓
Validate
        ↓
Publish result
```

The task shall have a higher priority than non-critical UI and web operations if required for timing reliability.

---

# 48. Storage Task

Responsibilities:

```text
Receive measurement record
        ↓
Write CSV
        ↓
Flush
        ↓
Report result
```

Storage operations should not block time-sensitive sensor operations.

---

# 49. UI Task

Responsibilities:

- Render screen.
- Process UI state.
- Update measurement values.
- Show errors.
- Show system status.

The UI shall not directly access sensor hardware.

---

# 50. Web Task

Responsibilities:

- Run HTTP server.
- Process REST requests.
- Serve static files.
- Request measurements.
- Read history.
- Manage configuration.

The web task shall communicate with application services.

---

# 51. Input Task

Responsibilities:

- Read encoder.
- Read buttons.
- Debounce inputs.
- Generate input events.

The input task should not directly perform measurements.

---

# 52. Concurrency Model

The preferred communication model is:

```text
                 ┌─────────────┐
                 │ Input Task  │
                 └──────┬──────┘
                        │
                        ▼
                  Input Queue
                        │
                        ▼
                Application
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
      Measurement Task          UI Task
             │
             ▼
      Measurement Queue
             │
       ┌─────┴─────┐
       ▼           ▼
 Storage Task    Web State
```

Shared state shall be protected using mutexes where required.

---

# 53. Thread Safety

Shared objects include:

- Current measurement.
- System state.
- Configuration.
- Sensor status.
- Storage status.

Access should use:

- Mutexes.
- Queues.
- Event groups.
- Atomic variables where appropriate.

Avoid using global mutable variables without synchronization.

---

# 54. Logging

The firmware should use ESP-IDF logging.

Recommended levels:

```text
ESP_LOGE
ESP_LOGW
ESP_LOGI
ESP_LOGD
```

Example:

```text
I (1234) EC: SEN0707 initialized
I (1250) TOF: HC-SR04 initialized
I (1300) STORAGE: SD mounted
W (1500) EC: Modbus timeout
E (1600) STORAGE: Failed to write record
```

Debug logging should be configurable.

---

# 55. Error Model

Errors shall be categorized.

```text
Hardware Errors
    ↓
Driver Errors
    ↓
Service Errors
    ↓
Application Errors
    ↓
User-visible Status
```

Example:

```text
MODBUS_TIMEOUT
      ↓
EC_SENSOR_ERROR
      ↓
MEASUREMENT_ERROR
      ↓
"EC SENSOR ERROR"
```

---

# 56. Watchdog Strategy

The ESP32-S3 watchdog mechanisms should be enabled according to ESP-IDF defaults and system requirements.

Tasks must not block indefinitely.

Long-running operations shall have:

- Timeouts.
- Recovery paths.
- Error handling.

---

# 57. Sensor Timeout Strategy

Every external sensor operation shall have a timeout.

Example:

```text
EC request
   ↓
Wait
   ↓
Response?
 ┌─┴─┐
Yes  No
 │    │
 ▼    ▼
OK   Retry
       │
       ▼
    Timeout
```

The system shall never wait indefinitely for a disconnected sensor.

---

# 58. Measurement Validation

Before a measurement is marked valid:

```text
TOF Valid?
    │
    ▼
Distance Valid?
    │
    ▼
EC Valid?
    │
    ▼
Values Within Expected Range?
    │
    ▼
Timestamp Valid?
    │
    ▼
Measurement VALID
```

Invalid values shall not be silently logged as valid measurements.

---

# 59. Sensor Disconnect Handling

The system shall detect communication failures.

For EC:

```text
No Modbus response
       ↓
Retry
       ↓
Sensor Error
```

For ultrasonic:

```text
No ECHO
   ↓
Timeout
   ↓
TOF Error
```

The display should identify which subsystem failed.

---

# 60. Firmware Update

V0.1 development shall initially use:

```text
USB
 ↓
ESP-IDF
 ↓
esptool
 ↓
ESP32-S3
```

OTA updates are not required for V0.1.

OTA may be added later through the local Wi-Fi interface.

---

# 61. Configuration Versioning

Configuration stored in NVS should include a version.

Example:

```cpp
#define CONFIG_VERSION 1
```

When configuration structures change, firmware should detect incompatible versions and restore safe defaults.

---

# 62. Application Initialization

Recommended initialization order:

```text
1. ESP32 startup
2. Logging
3. GPIO
4. SPI
5. I2C
6. UART
7. Timer/RMT
8. RTC
9. Display
10. SD
11. RS485
12. EC sensor
13. Ultrasonic sensor
14. Configuration
15. Application services
16. Wi-Fi
17. Web server
18. FreeRTOS tasks
19. READY
```

Initialization dependencies must be respected.

---

# 63. Startup Error Handling

A subsystem should be classified as either:

```text
Critical
Optional
```

Critical examples:

- ESP32 initialization
- Required GPIO
- Application services

Optional examples:

- SD card
- Wi-Fi
- RTC

The exact classification can be adjusted after prototype testing.

The device should provide useful diagnostics rather than simply failing to boot.

---

# 64. Memory Management

Avoid unnecessary dynamic allocation during active measurement.

Prefer:

- Static buffers.
- Fixed-size queues.
- Stack allocation for small temporary objects.
- Preallocated storage buffers.

Dynamic allocation is acceptable during initialization where practical.

---

# 65. Sensor Abstraction

The application shall use interfaces such as:

```text
IEcSensor
ITofSensor
```

This allows:

```text
                 Measurement Manager
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
          IEcSensor             ITofSensor
              │                     │
              ▼                     ▼
          SEN0707               HC-SR04
```

Future:

```text
          ITofSensor
              │
              ▼
       Dedicated TOF Driver
```

No major application rewrite should be required.

---

# 66. File Naming

Recommended source naming:

```text
snake_case.cpp
snake_case.h
```

Examples:

```text
measurement_manager.cpp
sen0707.cpp
storage_manager.cpp
web_server.cpp
```

Class names use PascalCase.

Examples:

```text
MeasurementManager
StorageManager
Sen0707
```

---

# 67. Constants

Hardware and application constants shall be centralized.

Example:

```cpp
namespace Config
{
    constexpr uint32_t EC_TIMEOUT_MS = 1000;
    constexpr uint8_t EC_RETRIES = 3;

    constexpr uint32_t TOF_TIMEOUT_US = 30000;

    constexpr uint32_t BUTTON_DEBOUNCE_MS = 50;
}
```

Avoid scattering magic numbers throughout the codebase.

---

# 68. Hardware Configuration

GPIO definitions should be centralized.

Example:

```cpp
namespace Pins
{
    constexpr int EC_TX = 17;
    constexpr int EC_RX = 18;
    constexpr int RS485_DE = 16;

    constexpr int TOF_TRIG = 4;
    constexpr int TOF_ECHO = 5;

    constexpr int ENCODER_A = 1;
    constexpr int ENCODER_B = 2;
    constexpr int ENCODER_SW = 3;

    constexpr int START = 38;
    constexpr int BACK = 39;
}
```

This makes hardware revisions easier.

---

# 69. Unit Conventions

Internal units shall be consistent.

Recommended:

| Measurement | Internal Unit |
|---|---|
| Conductivity | µS/cm |
| TOF | µs |
| Distance | mm |
| Temperature if used internally | °C |
| Battery | V |
| Time | UTC/local RTC representation |

The UI may convert units for presentation.

---

# 70. Conductivity Display

The internal conductivity value should remain in µS/cm.

The UI may automatically display:

```text
< 1000 µS/cm
```

as:

```text
820 µS/cm
```

and higher values as:

```text
4.82 mS/cm
```

The underlying stored value remains:

```text
4820 µS/cm
```

---

# 71. Measurement ID

Every completed measurement should receive a sequential ID.

Example:

```text
000001
000002
000003
```

The ID should help identify individual records.

The timestamp remains the primary time reference.

---

# 72. System Status

The system status should expose:

```text
System
├── MCU
├── Firmware Version
├── Uptime
│
├── EC Sensor
│   ├── Connected
│   └── Status
│
├── TOF Sensor
│   ├── Connected
│   └── Status
│
├── RTC
│   └── Status
│
├── SD
│   └── Status
│
├── Wi-Fi
│   └── Status
│
└── Battery
    └── Voltage
```

This information should be available through both TFT and web UI where practical.

---

# 73. Firmware Version

The firmware shall expose a version string.

Example:

```text
EC-TOF Analyzer Firmware
Version: 0.1.0
```

The version should be available through:

```text
TFT
Web API
Serial logs
```

---

# 74. Testing Architecture

The software should be testable at multiple levels.

```text
Unit Tests
    ↓
Driver Tests
    ↓
Integration Tests
    ↓
Hardware Tests
    ↓
System Tests
```

---

# 75. Driver-Level Testing

Test independently:

### EC

- Modbus request.
- Valid response.
- CRC error.
- Timeout.
- Invalid register.

### TOF

- Trigger.
- Echo.
- Timeout.
- Minimum distance.
- Maximum distance.

### RTC

- Read.
- Write.
- Backup behavior.

### SD

- Mount.
- Create file.
- Write.
- Read.
- Remove/reinsert.

---

# 76. Integration Testing

Test:

```text
EC + ESP32
TOF + ESP32
TFT + ESP32
SD + ESP32
RTC + ESP32
Wi-Fi + ESP32
```

Then:

```text
EC + TOF
EC + SD
TOF + SD
EC + TOF + TFT
EC + TOF + SD + RTC
```

Finally:

```text
Complete System
```

---

# 77. Software Acceptance Criteria

The V0.1 firmware is considered functionally complete when:

- ESP32-S3 boots reliably.
- TFT initializes reliably.
- User controls work.
- SEN0707 returns conductivity readings.
- RS485 communication handles errors.
- HC-SR04 produces valid prototype TOF measurements.
- Distance is calculated.
- DS3231 provides timestamps.
- Measurements are stored as CSV.
- SD errors are reported.
- Calibration values can be stored.
- Local Wi-Fi starts.
- Web dashboard loads.
- Measurement API works.
- Web measurement requests work.
- System status is available.
- Sensor failures are clearly reported.
- Firmware survives normal power cycling.

---

# 78. V0.1 Software Scope

Included:

```text
ESP-IDF firmware
SEN0707 Modbus driver
MAX3485 RS485 communication
HC-SR04 driver
TOF calculation
Distance calculation
ST7796 display
LVGL-based UI if practical
Rotary encoder
START button
BACK button
DS3231 RTC
MicroSD CSV logging
NVS configuration
EC calibration
TOF calibration
Wi-Fi AP
Local web server
REST API
System diagnostics
```

---

# 79. Explicit Software Exclusions

Not included in V0.1:

```text
Cloud backend
Mobile application
User accounts
Remote internet access
OTA firmware updates
AI processing
Cloud analytics
Database server
Advanced authentication
Production-grade ultrasonic DSP
Advanced signal processing
Multi-device synchronization
```

These may be considered in future versions.

---

# 80. Future Software Architecture

Potential future additions:

```text
Dedicated TOF processing
        ↓
Digital signal processing
        ↓
Advanced filtering
        ↓
Multi-point calibration
        ↓
Measurement profiles
        ↓
USB data export
        ↓
Advanced analytics
        ↓
OTA firmware
        ↓
Cloud synchronization
```

These features shall not complicate the V0.1 implementation unless required for the prototype.

---

# 81. Recommended Development Sequence

Software development should proceed in this order:

```text
1. ESP-IDF project
        ↓
2. GPIO and basic system
        ↓
3. TFT driver
        ↓
4. Input system
        ↓
5. RTC
        ↓
6. SD storage
        ↓
7. UART/RS485
        ↓
8. SEN0707 Modbus driver
        ↓
9. EC measurement
        ↓
10. HC-SR04 driver
        ↓
11. TOF measurement
        ↓
12. Measurement Manager
        ↓
13. Calibration
        ↓
14. Integrated UI
        ↓
15. Wi-Fi
        ↓
16. Web API
        ↓
17. Web UI
        ↓
18. Error handling
        ↓
19. System testing
```

---

# 82. Recommended First Firmware Milestone

The first firmware milestone should be:

```text
ESP32-S3
    +
TFT
    +
Rotary Encoder
    +
START / BACK
```

The target is to establish the basic user interface and hardware foundation before integrating sensors.

---

# 83. Recommended Sensor Milestone

The second major milestone:

```text
ESP32-S3
      │
      ├── MAX3485
      │       │
      │       └── SEN0707
      │
      └── HC-SR04
```

Target:

```text
EC Reading
+
TOF
+
Distance
```

shown simultaneously on the TFT.

---

# 84. Recommended Data Milestone

Third milestone:

```text
Measurement
     ↓
RTC Timestamp
     ↓
Measurement Record
     ↓
MicroSD
     ↓
CSV
```

Target example:

```text
2026-10-08T15:32:10,4820,12.482,18.73,VALID
```

---

# 85. Recommended Connectivity Milestone

Fourth milestone:

```text
ESP32-S3
    ↓
Wi-Fi AP
    ↓
Web Server
    ↓
REST API
    ↓
Browser
```

The web interface should consume the same measurement data used by the TFT.

---

# 86. Software Architecture Summary

The final V0.1 software architecture is:

```text
                         APPLICATION
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             ▼                ▼                ▼
       Measurement       Calibration      Configuration
         Manager           Manager           Manager
             │
             ▼
       Device Services
             │
    ┌────────┼────────┬────────┬────────┐
    ▼        ▼        ▼        ▼        ▼
    EC      TOF      RTC       SD      Input
    │        │        │        │        │
    ▼        ▼        ▼        ▼        ▼
 Modbus   Timing    I2C      SPI      GPIO
    │        │        │        │        │
    ▼        ▼        ▼        ▼        ▼
 SEN0707 HC-SR04 DS3231 MicroSD Controls

             ┌────────────────────────────┐
             │        User Interfaces     │
             │                            │
             │       TFT UI   Web UI      │
             └────────────────────────────┘
```

The measurement manager is the central application component.

Sensor drivers remain independent.

The UI, storage, and web interface consume validated measurement data rather than communicating directly with sensors.

---

# 87. Software Design Baseline

| Area | Decision |
|---|---|
| Framework | ESP-IDF |
| Language | C++ with ESP-IDF APIs |
| Architecture | Layered / modular |
| RTOS | FreeRTOS |
| EC Driver | SEN0707 Modbus RTU |
| RS485 | MAX3485 |
| TOF Driver | HC-SR04 |
| Timing | ESP32-S3 hardware timing peripheral |
| Display | ST7796 SPI |
| UI | LVGL where practical |
| Input | Rotary encoder + buttons |
| RTC | DS3231 |
| Storage | MicroSD + CSV |
| Configuration | NVS |
| Wi-Fi | Local AP |
| Web | ESP-IDF HTTP server |
| API | REST-style HTTP API |
| Calibration | EC + TOF |
| Measurement Core | State machine |
| Logging | Asynchronous where practical |
| Sensor Abstraction | Interfaces |
| Cloud | Not included |
| OTA | Not included |
| Database | Not included |

---

# 88. Implementation Rule

The most important software architecture rule is:

```text
UI must not control sensors directly.

Sensors must not control UI directly.

Storage must not control measurements directly.

Web API must not bypass application services.

Application services coordinate the system.
```

The intended dependency direction is:

```text
UI
 ↓
Application
 ↓
Device Services
 ↓
Drivers
 ↓
Hardware
```

Never reverse this dependency direction.

---

# 89. Final Software Structure

The final V0.1 firmware should conceptually follow:

```text
                  EC-TOF ANALYZER
                         │
                         ▼
                   App Manager
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
    Measurement     Calibration    Configuration
      Manager         Manager         Manager
          │
          ▼
    ┌─────┴───────────────────────────┐
    │                                 │
    ▼                                 ▼
Device Services                  User Interfaces
    │                                 │
 ┌──┼───────┬────────┐          ┌─────┴─────┐
 ▼  ▼       ▼        ▼          ▼           ▼
EC TOF     RTC       SD        TFT         Web
│   │       │        │
▼   ▼       ▼        ▼
RS485 RMT   I2C      SPI
│   │
▼   ▼
EC  HC-SR04
```

This architecture provides a clean separation between measurement hardware, application logic, storage, and user interfaces.

It is intentionally structured so the V0.1 prototype can evolve into a production system without requiring a complete firmware rewrite.