# EC-TOF Analyzer V0.1

Portable Electrical Conductivity and Ultrasonic Time-of-Flight Analyzer

---

## 1. Overview

The **EC-TOF Analyzer** is a portable, battery-powered measurement device designed to combine:

- Electrical conductivity measurement.
- Ultrasonic time-of-flight measurement.
- Distance calculation.
- Timestamped data logging.
- Local TFT display.
- Physical controls.
- Local Wi-Fi connectivity.
- REST API access.
- Rechargeable battery operation.
- Simple battery level indication.

V0.1 is a functional prototype.

The primary objective is to validate the complete measurement workflow and establish a modular hardware and software architecture that can later support a production-grade ultrasonic measurement system.

---

# 2. Project Status

**Version:** V0.1  
**Status:** Prototype Development  
**Platform:** ESP32-S3  
**Firmware:** ESP-IDF + C++  
**UI:** LVGL  
**Connectivity:** Local Wi-Fi  
**Storage:** MicroSD  
**Power:** Rechargeable Li-ion battery

### Current prototype approach

| Subsystem | V0.1 Implementation |
|---|---|
| Main controller | ESP32-S3-DevKitC-1-N8R8 |
| Conductivity | DFRobot SEN0707 |
| EC communication | RS485 / Modbus-RTU |
| RS485 transceiver | MAX3485 |
| Ultrasonic TOF | HC-SR04 |
| Display | 4" 480x320 ST7796 SPI TFT |
| RTC | DS3231 |
| Storage | MicroSD |
| User input | Rotary encoder + START + BACK |
| Battery | 3.7 V ~5000 mAh Li-ion |
| Charging | USB-C |
| SEN0707 supply | 12 V boost converter |
| Battery monitoring | ESP32 ADC voltage measurement |
| Battery indicator | Simple estimated percentage |
| Network | ESP32 local Wi-Fi AP |
| Firmware update | USB |

---

# 3. Important V0.1 Limitation

The **HC-SR04 is used only as a prototype ultrasonic sensor**.

It is an air ultrasonic ranging module.

It is **not** the final laboratory-grade ultrasonic measurement system.

V0.1 is intended to validate:

- TOF timing.
- Measurement sequencing.
- Distance calculation.
- Data processing.
- UI.
- Logging.
- System architecture.

A future version can replace the HC-SR04 with a dedicated ultrasonic transducer and receiver system without redesigning the entire application architecture.

---

# 4. Product Goals

The V0.1 prototype must allow the user to:

```text
Power ON
    │
    ▼
System READY
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
Measure Ultrasonic TOF
    │
    ▼
Calculate Distance
    │
    ▼
Measure Conductivity
    │
    ▼
Validate Measurement
    │
    ▼
Display Result
    │
    ▼
Timestamp Result
    │
    ▼
Log to MicroSD
    │
    ▼
Access Through Wi-Fi
    │
    ▼
READY
```

---

# 5. Core Features

## Electrical Conductivity

The analyzer uses the DFRobot SEN0707 industrial conductivity sensor.

Target specifications:

- K=10.
- 10 to 20,000 µS/cm.
- 1 µS/cm resolution.
- ±1% FS accuracy.
- RS485.
- Modbus-RTU.
- IP68 sensor construction.
- Built-in temperature compensation.

---

## Ultrasonic TOF

V0.1 uses an HC-SR04 for prototype development.

The system measures:

- Echo time.
- TOF.
- Calculated distance.

For the V0.1 air prototype:

```text
Distance = TOF × Speed of Sound / 2
```

The speed of sound is configurable.

The TOF subsystem is abstracted so the HC-SR04 can later be replaced.

---

## Display

The local UI uses a:

**4" 480x320 ST7796 SPI TFT**

The main screen displays:

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

The display is non-touch.

---

## Physical Controls

The device uses:

### Rotary Encoder

- Navigate menus.
- Select options.
- Adjust values.
- Confirm selections.

### START

Starts a measurement.

### BACK

Returns to the previous screen or cancels supported operations.

---

## Data Logging

Measurements are stored on MicroSD.

Example:

```csv
timestamp,conductivity,tof_us,distance_mm,status
2026-10-08T15:32:10,4820,12.482,18.73,VALID
```

Recommended storage structure:

```text
/ECTOF/
    config/
    data/
        2026/
            10/
                08.csv
    logs/
```

---

## Real-Time Clock

The DS3231 provides timestamps without requiring internet access.

Format:

```text
YYYY-MM-DDTHH:MM:SS
```

Example:

```text
2026-10-08T15:32:10
```

---

## Battery

The analyzer uses a rechargeable:

```text
3.7 V
~5000 mAh
Li-ion battery
```

Charging is provided through USB-C.

### Battery indicator

The system measures battery voltage using an ESP32 ADC and estimates the battery percentage.

Example:

```text
Battery: 78%
```

The battery indicator is an approximate user-facing estimate.

### Not included

V0.1 does **not** measure:

- Battery current.
- Battery power.
- Energy consumption.
- Detailed battery analytics.
- INA226.
- Dedicated fuel gauge.

---

# 6. System Architecture

```text
                         ┌───────────────────────┐
                         │       ESP32-S3        │
                         │   Main Controller     │
                         └───────────┬───────────┘
                                     │
       ┌─────────────────────────────┼─────────────────────────────┐
       │                             │                             │
       ▼                             ▼                             ▼
┌───────────────┐             ┌───────────────┐            ┌───────────────┐
│ EC Subsystem  │             │ TOF Subsystem │            │ User Interface│
│ SEN0707       │             │ HC-SR04       │            │ TFT / Controls│
│ RS485         │             │ Ultrasonic    │            │               │
└───────────────┘             └───────────────┘            └───────────────┘
       │                             │
       ▼                             ▼
   MAX3485                     GPIO + Level Shift
       │
       ▼
    RS485

       ┌─────────────────────────────┼─────────────────────────────┐
       │                             │                             │
       ▼                             ▼                             ▼
   MicroSD                        DS3231                       Wi-Fi
   Storage                          RTC                        Web/API

                         ┌───────────────────────┐
                         │    Power System       │
                         │ 3.7 V Li-ion Battery  │
                         └───────────────────────┘
```

---

# 7. Software Architecture

The firmware uses a layered architecture:

```text
┌────────────────────────────────────┐
│             UI Layer               │
│       LVGL TFT / Web UI            │
└──────────────────┬─────────────────┘
                   │
┌──────────────────▼─────────────────┐
│         Application Layer          │
│                                    │
│ Measurement Manager                │
│ Configuration Manager              │
│ Calibration Manager                │
│ System Manager                     │
│ Power Manager                      │
└──────────────────┬─────────────────┘
                   │
┌──────────────────▼─────────────────┐
│           Service Layer            │
│                                    │
│ EC Manager                         │
│ TOF Manager                        │
│ Storage Manager                    │
│ RTC Manager                        │
│ Wi-Fi Manager                      │
│ Web/API Manager                    │
└──────────────────┬─────────────────┘
                   │
┌──────────────────▼─────────────────┐
│            Driver Layer            │
│                                    │
│ RS485 / UART / SPI / I2C / GPIO    │
│ ADC / TFT / SD / RTC / Sensors     │
└────────────────────────────────────┘
```

---

# 8. Measurement State Machine

The measurement process follows:

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

Possible error paths include:

- TOF timeout.
- TOF sensor failure.
- EC timeout.
- EC communication error.
- Invalid measurement.
- SD logging failure.

---

# 9. Hardware Architecture

## Main Controller

**ESP32-S3-DevKitC-1-N8R8**

Responsibilities:

- Measurement control.
- Sensor communication.
- Display.
- Input.
- RTC.
- Storage.
- Battery ADC.
- Wi-Fi.
- Web API.
- Configuration.

---

## Conductivity

```text
ESP32-S3
    │
    ▼
UART
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

The EC sensor is connected through a removable external connector.

---

## Ultrasonic

```text
ESP32-S3
    │
    ├── TRIG ───────────────► HC-SR04
    │
    └── ECHO ◄── Level Shift ◄── HC-SR04
```

The ECHO signal must be level-shifted because the HC-SR04 can produce a 5 V output.

---

## Power

```text
3.7 V Battery
      │
      ├──► 12 V Boost ──► SEN0707
      │
      ├──► 5 V Rail ────► Peripherals
      │
      └──► 3.3 V Rail ──► ESP32 / Logic
```

Battery voltage is monitored through an ADC voltage divider.

---

# 10. Project Structure

Recommended firmware structure:

```text
ec-tof-analyzer/
│
├── CMakeLists.txt
├── sdkconfig.defaults
├── partitions.csv
├── README.md
│
├── main/
│   ├── main.cpp
│   │
│   ├── app/
│   │   ├── app_manager.*
│   │   ├── measurement_manager.*
│   │   ├── calibration_manager.*
│   │   ├── configuration_manager.*
│   │   └── system_manager.*
│   │
│   ├── sensors/
│   │   ├── ec/
│   │   └── tof/
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

The structure can be simplified during implementation if individual modules remain small.

---

# 11. Documentation

The project documentation is divided into the following documents:

| Document | Purpose |
|---|---|
| `01-product-requirements.md` | Product scope and requirements |
| `02-technical-requirements.md` | Detailed technical requirements |
| `03-system-architecture.md` | Overall system architecture |
| `04-hardware-design.md` | Hardware design and interfaces |
| `05-software-design.md` | Firmware architecture and software design |
| `06-implementation-plan.md` | Development and implementation phases |
| `README.md` | Project entry point |

---

# 12. Development Approach

The project follows a **prototype-first** approach.

Development should proceed incrementally.

### Phase 1: MCU Bring-Up

- ESP32-S3.
- ESP-IDF.
- GPIO.
- UART.
- SPI.
- I2C.
- ADC.

### Phase 2: EC Sensor

- MAX3485.
- RS485.
- Modbus-RTU.
- SEN0707.
- Conductivity reading.

### Phase 3: Ultrasonic

- HC-SR04.
- Trigger.
- Echo timing.
- TOF.
- Distance.

### Phase 4: User Interface

- TFT.
- LVGL.
- Rotary encoder.
- START.
- BACK.

### Phase 5: Measurement Manager

- Measurement state machine.
- Validation.
- Error handling.

### Phase 6: Storage

- DS3231.
- MicroSD.
- CSV logging.

### Phase 7: Power

- Battery.
- Charging.
- 12 V boost.
- Battery ADC.
- Battery indicator.

### Phase 8: Connectivity

- Wi-Fi AP.
- Web server.
- REST API.
- Web dashboard.

### Phase 9: Calibration

- EC calibration.
- TOF calibration.
- Configuration persistence.

### Phase 10: Validation

- Repeatability.
- Sensor failure.
- SD failure.
- Battery operation.
- Long-duration operation.
- Complete end-to-end workflow.

---

# 13. REST API

V0.1 provides:

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

Example system status:

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

---

# 14. Configuration

Configuration is stored using ESP-IDF NVS.

Potential configurable parameters include:

- EC Modbus slave ID.
- Conductivity limits.
- TOF limits.
- Distance limits.
- Speed of sound.
- Measurement timeout.
- Calibration parameters.
- Wi-Fi settings.
- Device settings.

Configuration must persist after reboot.

---

# 15. Calibration

## EC

EC calibration follows the SEN0707 manufacturer's procedure.

The supplied conductivity calibration solution can be used for verification.

## TOF

TOF calibration uses a known reference distance.

The calibration architecture must remain replaceable because the final ultrasonic hardware may use a different acoustic measurement model.

---

# 16. Error Handling

The system should detect and report:

```text
EC_TIMEOUT
EC_CRC_ERROR
EC_INVALID_RESPONSE
EC_DISCONNECTED

TOF_TIMEOUT
TOF_INVALID
TOF_DISCONNECTED

RTC_ERROR

SD_NOT_FOUND
SD_WRITE_ERROR

BATTERY_LOW
BATTERY_CRITICAL

CONFIG_INVALID
```

The exact error list may expand during implementation.

---

# 17. Reliability Principles

The firmware should:

- Use timeouts for external communication.
- Avoid indefinite blocking.
- Use watchdog protection.
- Retry recoverable sensor failures.
- Avoid infinite retry loops.
- Handle SD failure without crashing.
- Handle Wi-Fi disconnects.
- Protect shared state.
- Return the measurement system to a known state after failure.

---

# 18. V0.1 Exclusions

The following are intentionally outside the V0.1 scope:

- Production-grade liquid ultrasonic measurement.
- Dedicated ultrasonic analog front end.
- Advanced acoustic signal processing.
- Battery current measurement.
- Battery power measurement.
- INA226.
- Dedicated fuel gauge.
- Detailed battery analytics.
- Cloud backend.
- Internet connectivity.
- Mobile application.
- Web OTA.
- GPS.
- Cellular connectivity.
- Touchscreen.
- Automated sample identification.
- Industrial certification.
- Production EMC certification.
- Final production enclosure certification.

---

# 19. Future Development

Potential V0.2 and production improvements include:

### Ultrasonic

- Laboratory-grade transducer.
- Dedicated receiver.
- Analog front end.
- Precision timing.
- Acoustic signal processing.
- Improved sample cell.
- Temperature compensation.

### Electrical Conductivity

- Alternative laboratory-grade EC sensor.
- Improved sample cell.
- Additional calibration options.

### Hardware

- Custom PCB.
- Improved power architecture.
- Better connectors.
- Improved enclosure.
- Improved EMI/EMC design.

### Software

- Advanced acoustic processing.
- More calibration options.
- Measurement analytics.
- OTA updates.
- User authentication.
- Optional cloud integration.

---

# 20. Definition of Done

V0.1 is considered complete when the device can perform the complete workflow:

```text
┌───────────────────────────────┐
│           POWER ON            │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│       SYSTEM INITIALIZE       │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│             READY             │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│        PRESS START            │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│       MEASURE ULTRASONIC      │
│             TOF               │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│       CALCULATE DISTANCE      │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│       MEASURE CONDUCTIVITY    │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│       VALIDATE RESULT         │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│        DISPLAY RESULT         │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│        LOG TO MICROSD         │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│        WEB INTERFACE          │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│             READY             │
└───────────────────────────────┘
```

---

# 21. Final V0.1 Architecture

```text
                    EC-TOF ANALYZER V0.1
                            │
                            ▼
                    ┌───────────────┐
                    │   ESP32-S3    │
                    └───────┬───────┘
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
       EC SYSTEM        TOF SYSTEM        USER INTERFACE
          │                 │                 │
          ▼                 ▼                 ▼
      SEN0707            HC-SR04          ST7796 TFT
          │                 │             Encoder
       MAX3485             │             START/BACK
          │                 │
         RS485          GPIO + Level
                         Shifting

          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
       MicroSD            DS3231            Wi-Fi
       Logging              RTC             Web/API

                            │
                            ▼
                    ┌───────────────┐
                    │    Battery    │
                    │  3.7 V Li-ion │
                    └───────────────┘
                            │
                            ▼
                    Simple Battery %
```

---

# 22. Project Principle

The EC-TOF Analyzer V0.1 follows one primary development principle:

> **Prove the measurement workflow first, then optimize the hardware for production.**

The prototype intentionally uses readily available components while keeping the software and system architecture modular.

The most important future-proofing feature is the abstraction of the sensor interfaces.

The EC sensor and ultrasonic sensor can therefore be replaced without requiring a complete rewrite of the application.

---

## Project Documentation

Start with the documents in this order:

```text
01-product-requirements.md
        │
        ▼
02-technical-requirements.md
        │
        ▼
03-system-architecture.md
        │
        ▼
04-hardware-design.md
        │
        ▼
05-software-design.md
        │
        ▼
06-implementation-plan.md
        │
        ▼
       CODE
```

This README provides the high-level project overview.

The detailed requirements and implementation decisions are defined in the individual project documents.