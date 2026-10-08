# EC-TOF Analyzer V0.1
## Technical Requirements

**Version:** 0.1  
**Status:** Prototype  
**Target Platform:** ESP32-S3  
**Firmware:** ESP-IDF  
**Last Updated:** October 2026

---

# 1. Purpose

This document defines the technical requirements for the EC-TOF Analyzer V0.1 prototype.

The system shall:

- Measure electrical conductivity of a liquid sample.
- Measure ultrasonic time of flight.
- Calculate distance from the measured TOF.
- Timestamp measurements.
- Store measurement records on a MicroSD card.
- Display measurements locally.
- Provide a local Wi-Fi web interface.
- Operate from a rechargeable battery.
- Provide a simple battery level indicator.
- Support removable external sensors.
- Provide physical controls.
- Use modular firmware architecture.
- Allow future replacement of prototype sensors with production-grade sensors.

V0.1 is a feasibility and prototype platform. It is not intended to be a final laboratory-grade measurement instrument.

---

# 2. System Constraints

## 2.1 Controller

The system shall use:

- ESP32-S3-DevKitC-1-N8R8
- ESP-IDF
- C++ application code where practical
- FreeRTOS through ESP-IDF
- Modular drivers and application services

---

## 2.2 External Sensor Interface

External sensors shall use interfaces that are appropriate for electrically noisy environments.

The system shall avoid using I2C for external sensors.

### External interfaces

| Device | Interface |
|---|---|
| SEN0707 conductivity sensor | RS485 / Modbus RTU |
| HC-SR04 ultrasonic sensor | GPIO trigger/echo |
| Display | SPI |
| MicroSD | SPI |
| DS3231 RTC | I2C |
| Battery measurement | ESP32 ADC |
| Rotary encoder | GPIO |
| Buttons | GPIO |

I2C is acceptable for short internal connections such as the RTC.

---

# 3. Electrical Conductivity Requirements

## 3.1 Sensor

The prototype shall use the:

**DFRobot SEN0707 Industrial Water Conductivity Sensor**

Key characteristics:

- Measurement range: 10 to 20,000 µS/cm
- Resolution: 1 µS/cm
- Accuracy: ±1% FS
- Cell constant: K=10
- Interface: RS485
- Protocol: Modbus RTU
- Supply: 10 to 30 V DC
- Typical power: approximately 0.4 W
- Protection: IP68
- Built-in temperature compensation

---

## 3.2 EC Communication

The ESP32-S3 shall communicate with the SEN0707 through an RS485 transceiver.

Required path:

```text
ESP32-S3 UART
      │
      ▼
MAX3485
      │
      ▼
RS485
      │
      ▼
SEN0707
```

The firmware shall support:

- Modbus RTU requests
- Response parsing
- CRC validation
- Communication timeout
- Invalid response detection
- Sensor disconnect detection
- Communication error reporting

---

## 3.3 EC Measurement Data

The application layer shall receive EC measurements in a standardized internal format.

The system shall support:

- Raw EC value
- Engineering unit
- Sensor status
- Communication status
- Measurement timestamp

The preferred display unit is:

**µS/cm**

The UI may additionally display:

**mS/cm**

where appropriate.

---

# 4. Ultrasonic TOF Requirements

## 4.1 Prototype Sensor

V0.1 shall use an:

**HC-SR04 ultrasonic sensor**

The HC-SR04 is used only for prototype feasibility testing.

It shall not be considered the final acoustic measurement hardware.

---

## 4.2 TOF Measurement

The firmware shall measure:

- Trigger time
- Echo start
- Echo end
- Echo pulse duration
- Calculated TOF
- Calculated distance
- Measurement validity

For the air-based prototype:

```text
Distance = TOF × Speed of Sound / 2
```

The firmware shall keep the TOF implementation abstract so the HC-SR04 can later be replaced with a different ultrasonic/acoustic sensor.

---

## 4.3 Echo Protection

The HC-SR04 ECHO output may exceed the ESP32-S3 GPIO voltage level.

Therefore:

```text
HC-SR04 ECHO
      │
      ▼
Voltage Divider / Level Shifter
      │
      ▼
ESP32-S3 GPIO
```

The ESP32-S3 GPIO shall never be directly exposed to an unsafe voltage.

---

# 5. Display Requirements

## 5.1 Display

The prototype shall use:

- 4-inch TFT
- 480 × 320 resolution
- ST7796 controller
- SPI interface
- Non-touch operation

The display shall use a removable connector.

---

## 5.2 Main Display

The main screen should show:

```text
EC-TOF ANALYZER

Conductivity: 4.82 mS/cm
TOF:          12.482 us
Distance:     18.73 mm

Status: READY

Battery: 78%
```

The exact UI layout may change during implementation.

---

## 5.3 Required Screens

The firmware shall support at least:

1. Main Screen
2. Measurement Screen
3. Calibration Screen
4. Configuration Screen
5. System Status Screen

---

# 6. Physical Controls

The system shall use physical controls instead of a touchscreen.

Required controls:

- Rotary encoder
- Encoder push button
- START button
- BACK button

The rotary encoder shall support:

- Menu navigation
- Value adjustment
- Configuration selection

The push button shall support:

- Menu selection
- Confirmation

START shall initiate a measurement.

BACK shall return to the previous screen or cancel an operation.

---

# 7. RTC Requirements

The system shall use:

**DS3231 RTC**

Interface:

**I2C**

The RTC shall provide:

- Date
- Time
- Timestamp for measurements
- Timestamp for log files

Required timestamp format:

```text
YYYY-MM-DDTHH:MM:SS
```

The RTC shall continue operating independently of Wi-Fi availability.

---

# 8. Data Logging Requirements

## 8.1 Storage

The system shall use:

- MicroSD card
- SPI interface
- 16 GB or 32 GB recommended for V0.1

---

## 8.2 File Structure

Recommended structure:

```text
/ECTOF/
    config/
    data/
        YYYY/
            MM/
                DD.csv
    logs/
```

---

## 8.3 Measurement Log

CSV records shall contain:

```text
timestamp,conductivity,tof_us,distance_mm,status
```

Example:

```text
2026-10-08T15:32:10,4820,12.482,18.73,VALID
```

The logging system shall:

- Create files when required.
- Append measurement records.
- Detect SD errors.
- Report storage failures.
- Prevent SD errors from crashing the application.

---

# 9. Battery Requirements

## 9.1 Battery

V0.1 shall use:

- 3.7 V rechargeable LiPo/Li-ion battery
- Approximately 5000 mAh capacity

The battery shall include appropriate protection.

---

## 9.2 Charging

The system shall provide:

- USB-C charging
- 1-cell Li-ion/LiPo charging
- Appropriate charging protection

A power-path charger may be used if simultaneous charging and operation is required.

---

## 9.3 Power Rails

The prototype shall provide appropriate regulated rails for the system components.

Recommended architecture:

```text
3.7 V Battery
      │
      ▼
Power Distribution
      │
      ├── 12 V Boost ──► SEN0707
      │
      ├── 5 V Regulator ──► TFT / HC-SR04
      │
      └── 3.3 V Regulator ──► ESP32-S3
                                  │
                                  ├── RTC
                                  ├── MicroSD
                                  └── MAX3485
```

Actual regulator selection shall be finalized during hardware implementation.

---

# 10. Battery Level Indicator

V0.1 shall provide a simple battery level indicator.

The system shall measure battery voltage using an ESP32-S3 ADC.

Architecture:

```text
Battery +
   │
   ▼
Resistor Voltage Divider
   │
   ▼
ESP32-S3 ADC
   │
   ▼
Battery Voltage
   │
   ▼
Estimated Battery %
```

The voltage divider shall scale the battery voltage to a safe ESP32-S3 ADC input range.

---

## 10.1 Battery Percentage

The battery percentage shall be an estimated value based on battery voltage.

It shall not be treated as a precision state-of-charge measurement.

A simple lookup table may be used.

Example:

| Battery Voltage | Estimated Level |
|---:|---:|
| ≥ 4.10 V | 100% |
| 4.00–4.09 V | 80% |
| 3.90–3.99 V | 60% |
| 3.80–3.89 V | 50% |
| 3.70–3.79 V | 35% |
| 3.60–3.69 V | 20% |
| 3.50–3.59 V | 10% |
| ≤ 3.40 V | Critical |

The exact lookup table may be adjusted during prototype testing.

---

## 10.2 Battery UI

The display shall provide a simple indicator such as:

```text
🔋 78%
```

or an equivalent graphical battery icon.

The web interface shall also expose:

- Battery voltage
- Estimated battery percentage
- Battery status

The system shall not perform battery current measurement in V0.1.

The system shall not calculate battery power consumption in V0.1.

---

# 11. Wi-Fi Requirements

The ESP32-S3 shall provide a local Wi-Fi access point.

Recommended default:

```text
SSID: EC-TOF-Analyzer
IP:   192.168.4.1
```

The configuration may be changed later.

The device shall provide a local web interface.

---

# 12. Web Interface Requirements

The web interface shall provide:

## Dashboard

Display:

- Current EC
- Current TOF
- Current distance
- Measurement status
- Battery voltage
- Estimated battery percentage
- Device status

## Measurements

Display:

- Recent measurements
- Measurement timestamp
- EC
- TOF
- Distance
- Status

## Calibration

Provide access to:

- EC calibration
- TOF calibration

## Configuration

Provide access to:

- Measurement settings
- Device settings
- Logging settings

## System Status

Display:

- Firmware version
- Device uptime
- RTC status
- SD status
- RS485 status
- Sensor status
- Battery voltage
- Estimated battery percentage

---

# 13. REST API Requirements

The device shall expose a local REST API.

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

The API shall return structured JSON responses.

---

# 14. Measurement Sequence

The primary measurement workflow shall be:

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

The system shall provide error paths for:

- TOF timeout
- EC timeout
- RS485 communication failure
- Invalid EC response
- Invalid TOF measurement
- SD failure
- RTC failure

---

# 15. Configuration Requirements

Configuration shall be stored using ESP-IDF NVS.

Configuration should include:

- Device settings
- EC settings
- TOF settings
- Calibration parameters
- Logging settings
- Display settings

Configuration shall survive a reboot.

Configuration data shall be versioned to support future firmware updates.

---

# 16. Calibration Requirements

## 16.1 EC Calibration

EC calibration shall follow the SEN0707 manufacturer's recommended procedure.

The supplied conductivity calibration solution may be used for prototype verification.

The calibration system shall support:

- Calibration initiation
- Calibration value entry
- Calibration result validation
- Saving calibration parameters
- Loading calibration parameters after reboot

---

## 16.2 TOF Calibration

TOF calibration shall use a known reference distance.

The system shall allow:

- Reference distance entry
- Measurement collection
- Offset or correction calculation
- Saving calibration parameters

The calibration architecture shall remain independent of the HC-SR04 implementation.

---

# 17. Software Architecture Requirements

The firmware shall be modular.

Recommended structure:

```text
main/
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

Sensor drivers shall not directly control the display.

The web interface shall not directly communicate with hardware drivers.

The application layer shall coordinate system behavior.

---

# 18. Error Handling

The system shall detect and report hardware and software errors.

Required error categories:

### EC

- Sensor disconnected
- RS485 timeout
- CRC error
- Invalid Modbus frame
- UART error

### TOF

- No echo
- Timeout
- Out-of-range result
- Invalid measurement

### SD

- Card missing
- Mount failure
- Write failure
- File failure

### RTC

- Communication failure
- Invalid time

### Battery

- ADC failure
- Out-of-range voltage
- Critical battery voltage

Errors shall be displayed in a user-readable form.

Errors shall also be available through the web interface.

---

# 19. Safety Requirements

The prototype shall include appropriate electrical protection.

Recommended:

- Battery protection
- Input fuse or polyfuse
- Reverse-polarity protection where appropriate
- Protected voltage rails
- Proper connector ratings
- ESP32 GPIO voltage protection
- HC-SR04 ECHO level shifting
- Proper enclosure
- Strain relief for external sensors

The 12 V SEN0707 supply shall be isolated from unsafe battery voltage conditions through a properly rated regulator or boost converter.

---

# 20. Replaceability Requirements

The following components shall be replaceable without redesigning the complete application:

- EC sensor
- TOF sensor
- Display
- SD module
- RTC
- Battery
- Power converter

The software shall use interfaces/abstractions for sensor implementations.

Example:

```cpp
class IEcSensor
{
public:
    virtual bool begin() = 0;
    virtual bool read(float& conductivity) = 0;
    virtual bool calibrate() = 0;
};
```

TOF shall follow a similar abstraction.

---

# 21. Prototype Acceptance Criteria

V0.1 shall be considered technically successful when:

- ESP32-S3 boots reliably.
- TFT displays the application UI.
- Rotary encoder and buttons work.
- SEN0707 communicates reliably over RS485.
- EC values can be read.
- HC-SR04 can produce repeatable prototype distance measurements.
- TOF can be measured.
- Distance can be calculated.
- DS3231 provides valid timestamps.
- Measurements can be saved to MicroSD.
- Local Wi-Fi starts successfully.
- Web dashboard displays measurement data.
- Battery voltage can be measured through the ADC.
- A simple estimated battery percentage can be displayed.
- External sensors can be disconnected and replaced.
- Sensor failures do not crash the application.

---

# 22. Explicit V0.1 Exclusions

The following are intentionally excluded from V0.1:

- Precision battery state-of-charge measurement
- Battery current measurement
- Battery power measurement
- Dedicated battery fuel gauge
- Industrial-grade ultrasonic/acoustic transducer
- Final liquid acoustic TOF measurement method
- Cellular connectivity
- Cloud connectivity
- Remote monitoring
- Touchscreen interface
- OTA firmware updates
- Advanced power analytics
- Production certification

These may be considered in future versions after the prototype architecture has been validated.

---

# 23. Future V0.2 / Production Considerations

Future versions may introduce:

- Production-grade acoustic transducer
- Dedicated ultrasonic receiver electronics
- Improved TOF timing hardware
- More accurate battery fuel gauge
- Advanced battery management
- USB data export
- OTA firmware updates
- Improved enclosure
- Waterproof external connectors
- Calibration history
- User accounts
- Cloud synchronization
- Advanced measurement analytics

These features shall not complicate the V0.1 implementation unless they are required for validating the core measurement concept.