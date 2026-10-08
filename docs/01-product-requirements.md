# EC-TOF Analyzer
## Product Requirements Document

Version: 0.1  
Status: Prototype Planning  
Product Type: Portable Laboratory Measurement Instrument

---

## 1. Product Overview

The EC-TOF Analyzer is a portable laboratory instrument designed to measure:

1. Electrical conductivity of a liquid sample
2. Ultrasonic time of flight
3. Calculated distance based on the ultrasonic measurement

The first prototype will use an ESP32-S3 as the main controller.

The prototype will use commercially available components wherever possible. Custom electronics will be avoided during the initial development stage.

The system will provide a local display, data logging, battery operation, and a local web interface.

---

## 2. Product Objective

The primary objective is to build a working prototype that can demonstrate the customer's required measurement concept using low-cost, readily available hardware.

The prototype must:

- Measure electrical conductivity
- Measure ultrasonic time of flight
- Calculate distance
- Display measurement results
- Store measurement data
- Timestamp measurements
- Operate from a rechargeable battery
- Allow the user to start and control measurements without a touchscreen
- Provide a local web interface for viewing and exporting data

The prototype is intended for engineering validation and experimentation.

It is not considered a production laboratory instrument until the measurement accuracy and sensor technology have been validated.

---

## 3. V0.1 Scope

### 3.1 Included

The first prototype will include:

- ESP32-S3 controller
- DFRobot SEN0707 conductivity sensor
- RS485 communication
- Modbus RTU communication with the conductivity sensor
- HC-SR04 ultrasonic sensor
- 4-inch SPI TFT display
- MicroSD data logging
- DS3231 RTC
- Physical user controls
- Rechargeable battery
- Battery charging
- Battery monitoring
- Local Wi-Fi
- Local web interface
- Measurement history
- CSV data export
- Basic calibration functions

---

## 4. Measurements

### 4.1 Electrical Conductivity

The system will use the DFRobot SEN0707 industrial conductivity sensor.

The sensor communicates with the ESP32-S3 through RS485 using Modbus RTU.

The system will display conductivity using an appropriate unit such as:

- µS/cm
- mS/cm

The initial supported measurement range will follow the SEN0707 specifications.

The prototype will not implement a custom conductivity analog front end.

### 4.2 Ultrasonic Time of Flight

The first prototype will use an HC-SR04 ultrasonic sensor.

The ESP32-S3 will:

1. Trigger the ultrasonic sensor.
2. Start a high-resolution timer.
3. Monitor the echo signal.
4. Measure the elapsed time.
5. Store the measured time of flight.
6. Calculate distance where applicable.

The HC-SR04 is a prototype sensor only.

It must not be considered the final ultrasonic sensor for production use.

### 4.3 Distance

Distance will be calculated from the ultrasonic measurement.

The initial implementation will use the ultrasonic sensor's measured echo time and the configured acoustic propagation parameters.

The calculation method may change after laboratory testing determines the customer's required ultrasonic measurement configuration.

---

## 5. User Requirements

The instrument should be simple enough for a laboratory operator to use without a computer.

The operator must be able to:

- Power the instrument on
- View the current system status
- Start a measurement
- View conductivity
- View ultrasonic time of flight
- View calculated distance
- View measurement status
- Review previous measurements
- Start calibration procedures
- Configure basic system settings
- Export stored measurements

The instrument will use physical controls rather than a touchscreen.

---

## 6. User Interface Requirements

The primary interface will be a 4-inch SPI TFT display.

The initial interface should display:

```text
EC-TOF ANALYZER

Conductivity
4.82 mS/cm

Ultrasonic TOF
12.482 µs

Distance
18.73 mm

Status
VALID

Battery
82%
```

The interface should prioritize measurement values over decorative graphics.

The system should use clear status indicators for:

- Ready
- Measuring
- Valid
- Error
- Sensor disconnected
- Calibration required
- Low battery
- SD card error

---

## 7. Physical Controls

The prototype will use:

- Rotary encoder with push button
- START button
- BACK button

The rotary encoder will be used for:

- Menu navigation
- Value selection
- Configuration
- Calibration input

The push button on the encoder will be used for:

- Select
- Enter
- Confirm

The START button will begin a measurement.

The BACK button will return to the previous menu or cancel an operation.

---

## 8. Data Logging Requirements

The system must store measurement data on a MicroSD card.

Each measurement record should contain at least:

- Measurement ID
- Timestamp
- Conductivity
- Ultrasonic time of flight
- Calculated distance
- Measurement status

Example:

```text
id,timestamp,conductivity,tof_us,distance_mm,status
1,2026-10-08T15:32:10,4820,12.482,18.73,VALID
2,2026-10-08T15:33:10,4818,12.479,18.72,VALID
```

CSV will be the initial data format.

The system should continue operating if an SD card is not installed, but data logging must be reported as unavailable.

---

## 9. Time and Timestamp Requirements

The prototype will use a DS3231 RTC.

The RTC will provide timestamps when recording measurements.

The instrument must maintain the correct date and time across power cycles using the RTC backup battery.

The system should allow the user to configure the date and time.

Internet connectivity must not be required for timestamping.

---

## 10. Battery Requirements

The instrument must operate from a rechargeable battery.

The prototype will use a single-cell rechargeable lithium battery.

The battery system must provide the required voltage rails for:

- ESP32-S3
- Display
- HC-SR04
- SEN0707

Because the SEN0707 requires a higher supply voltage than the ESP32-S3, the prototype will use a boost converter to generate its required supply voltage.

The system must provide:

- Battery charging
- Main power switch
- Battery status
- Low-battery warning
- Basic battery protection

The system should display the approximate battery level on the main screen.

---

## 11. Connectivity Requirements

### 11.1 Sensor Connectivity

The conductivity sensor will communicate through:

```text
ESP32-S3
    ↓
UART
    ↓
RS485
    ↓
SEN0707
```

The communication protocol will be Modbus RTU.

The ultrasonic sensor will use GPIO signals.

### 11.2 Wi-Fi

The ESP32-S3 will provide local Wi-Fi connectivity.

The instrument should be capable of creating a local access point.

Example:

```text
SSID:
EC-TOF-Analyzer

IP:
192.168.4.1
```

Internet access must not be required for normal operation.

---

## 12. Web Interface Requirements

The local web interface should allow an operator to:

- View current measurements
- View device status
- View measurement history
- Configure basic settings
- Configure calibration parameters
- Download CSV measurement files

Initial pages:

```text
Dashboard
Measurements
Calibration
Configuration
Data Export
System Status
```

The web interface is intended for local use.

Cloud connectivity is outside the V0.1 scope.

---

## 13. Calibration Requirements

The system must support basic calibration.

### Conductivity

The SEN0707 will be calibrated according to the manufacturer's recommended procedure.

The system should provide a calibration workflow for entering or verifying the calibration condition.

### Ultrasonic

The prototype must provide a method to configure the parameters required for distance calculation.

A known reference distance should be used during prototype validation.

The ultrasonic calibration approach may be revised after testing.

---

## 14. Sensor Connection Requirements

External sensors must be removable.

The conductivity sensor must not be permanently soldered to the controller.

The ultrasonic sensor must also use a removable connection.

The prototype should use panel-mounted connectors where practical.

The intended architecture is:

```text
Controller
    │
    ├── Removable EC connector
    │
    └── Removable ultrasonic connector
```

This allows the prototype sensors to be replaced later without redesigning the controller.

---

## 15. Prototype Hardware Requirements

The prototype should use off-the-shelf components.

The initial hardware should include:

- ESP32-S3 development board
- DFRobot SEN0707
- RS485 transceiver
- HC-SR04
- 4-inch SPI TFT
- MicroSD module
- MicroSD card
- DS3231
- Rotary encoder
- START button
- BACK button
- Rechargeable battery
- Battery charger
- DC-DC converters
- Connectors
- Prototype enclosure

A custom PCB is not required for V0.1.

---

## 16. Software Requirements

The firmware will be developed using ESP-IDF.

The software should be modular.

Major software components should include:

```text
System
Sensors
 ├── EC
 └── Ultrasonic
Display
Input
Storage
RTC
Calibration
Power
Wi-Fi
Web
Configuration
```

Sensor drivers must be separated from application logic.

The application should not directly depend on low-level sensor communication code.

---

## 17. Reliability Requirements

The prototype should handle common failure conditions.

The system must detect and report:

- EC sensor communication failure
- EC sensor disconnected
- Ultrasonic timeout
- Invalid ultrasonic measurement
- SD card unavailable
- RTC communication failure
- Low battery
- Invalid configuration
- Wi-Fi initialization failure

A sensor failure must not cause the entire application to crash.

---

## 18. Safety Requirements

The prototype must:

- Use a protected rechargeable battery
- Include appropriate battery charging protection
- Prevent direct 5 V HC-SR04 signals from entering ESP32-S3 GPIOs
- Use appropriate fusing or current protection
- Keep the higher-voltage SEN0707 supply isolated from ESP32 GPIO signals
- Provide strain relief for external sensor cables
- Avoid exposed conductive connections during normal operation

The prototype is intended for laboratory development and must not be treated as a certified measurement or safety instrument.

---

## 19. V0.1 Acceptance Criteria

The V0.1 prototype will be considered functional when:

### Controller

- ESP32-S3 boots reliably.
- Firmware can be updated through USB.
- Main UI starts successfully.

### Conductivity

- SEN0707 communicates successfully through RS485.
- Conductivity values can be read by the ESP32-S3.
- Conductivity is displayed on the TFT.

### Ultrasonic

- HC-SR04 can be triggered.
- Echo time can be measured.
- TOF is displayed.
- Distance is calculated.
- Invalid or timeout conditions are detected.

### Display

- Measurement values are clearly displayed.
- Physical controls operate correctly.
- System status is visible.

### Storage

- SD card can be initialized.
- Measurement records can be written.
- CSV files can be generated.
- Stored data can be retrieved.

### RTC

- Date and time can be configured.
- Measurements receive timestamps.
- Time is retained after power loss.

### Battery

- System operates from the battery.
- Battery can be charged.
- Battery level can be displayed.
- Low battery condition is detected.

### Web Interface

- ESP32-S3 can create a local Wi-Fi network.
- Dashboard can be accessed from a phone or computer.
- Current measurements can be viewed.
- CSV data can be downloaded.

---

## 20. Out of Scope for V0.1

The following are explicitly excluded from the first prototype:

- Production-grade ultrasonic transducer system
- Custom ultrasonic analog electronics
- Custom conductivity electronics
- Cloud backend
- Mobile application
- User accounts
- Remote monitoring
- Internet-dependent operation
- Automatic firmware updates
- Touchscreen interface
- AI features
- Temperature as a user-facing measurement
- TDS as a user-facing measurement
- Salinity as a user-facing measurement
- Industrial certification
- Production PCB
- Production enclosure
- Laboratory certification

These features may be considered in later versions if required.

---

## 21. Future Development

Potential future improvements include:

- Dedicated ultrasonic TX/RX transducers
- Improved ultrasonic receiver electronics
- Higher accuracy TOF measurement
- Custom controller PCB
- Industrial connectors
- Improved battery management
- Larger storage
- USB data export
- Advanced calibration
- Measurement reports
- Automated test procedures
- Production-grade enclosure
- Laboratory validation
- Measurement accuracy certification

Future features must only be added when they provide a clear benefit to the customer's requirements.

---

## 22. Product Development Principle

The project will follow an off-the-shelf-first approach.

The first prototype should prove the measurement concept before custom hardware is developed.

The development sequence is:

```text
Off-the-shelf hardware
        ↓
Electrical integration
        ↓
Firmware integration
        ↓
Measurement testing
        ↓
Calibration
        ↓
Prototype validation
        ↓
Identify limitations
        ↓
Upgrade hardware
        ↓
Production design
```

The HC-SR04 is specifically considered a feasibility sensor.

The final ultrasonic technology will be selected only after the prototype demonstrates the customer's actual measurement requirements.

---

## 23. V0.1 Product Definition

The V0.1 EC-TOF Analyzer is defined as:

> A portable, battery-powered ESP32-S3 laboratory prototype capable of measuring electrical conductivity using the DFRobot SEN0707, measuring ultrasonic time of flight using an HC-SR04, calculating distance, displaying measurements locally, recording measurements to a MicroSD card, timestamping measurements using an RTC, and providing a local Wi-Fi web interface.

This definition is the baseline for the remaining technical, hardware, software, and implementation documents.