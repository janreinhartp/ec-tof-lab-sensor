# EC-TOF Analyzer V0.1
# Hardware Design

## 1. Purpose

This document defines the hardware design for the EC-TOF Analyzer V0.1 prototype.

The hardware must support:

- Electrical conductivity measurement.
- Ultrasonic time-of-flight measurement.
- Distance calculation.
- Local measurement display.
- Measurement logging.
- Real-time timestamps.
- Local Wi-Fi connectivity.
- Battery-powered operation.
- Rechargeable battery.
- Simple battery level indication.
- Physical user controls.
- Removable external sensors.

V0.1 is a functional prototype.

The hardware is designed to prove the measurement workflow before moving to a production-grade ultrasonic measurement system.

---

# 2. Hardware Architecture

The system uses an ESP32-S3 as the main controller.

```text
                         ┌──────────────────────┐
                         │      ESP32-S3        │
                         │   Main Controller    │
                         └──────────┬───────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
        ▼                           ▼                           ▼
┌───────────────┐           ┌───────────────┐          ┌───────────────┐
│ EC Sensor     │           │ TOF Sensor    │          │ TFT Display   │
│ SEN0707       │           │ HC-SR04       │          │ ST7796        │
│ RS485         │           │ Ultrasonic    │          │ SPI           │
└───────┬───────┘           └───────────────┘          └───────────────┘
        │
        ▼
┌───────────────┐
│ MAX3485       │
│ RS485         │
└───────────────┘

        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
        ▼                           ▼                           ▼
┌───────────────┐           ┌───────────────┐          ┌───────────────┐
│ MicroSD       │           │ DS3231 RTC    │          │ User Controls │
│ SPI           │           │ I2C           │          │ Encoder       │
└───────────────┘           └───────────────┘          │ START / BACK  │
                                                       └───────────────┘

                         ┌──────────────────────┐
                         │ Battery / Power      │
                         │ 3.7 V Li-ion        │
                         └──────────┬───────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              ▼                     ▼                     ▼
         12 V Boost              5 V Rail              3.3 V Rail
              │                     │                     │
              ▼                     ▼                     ▼
         SEN0707               TFT / HC-SR04          ESP32-S3
```

---

# 3. Main Controller

## 3.1 ESP32-S3-DevKitC-1-N8R8

The ESP32-S3-DevKitC-1-N8R8 is the primary controller.

### Responsibilities

- Sensor communication.
- Measurement sequencing.
- TOF timing.
- Distance calculation.
- Conductivity acquisition.
- Display control.
- Physical input handling.
- RTC communication.
- SD card logging.
- Battery voltage measurement.
- Battery percentage estimation.
- Wi-Fi access point.
- REST API.
- Configuration management.
- Calibration management.
- System status management.

### Recommended configuration

| Parameter | Specification |
|---|---|
| MCU | ESP32-S3 |
| Flash | 8 MB |
| PSRAM | 8 MB |
| Firmware | ESP-IDF |
| Logic voltage | 3.3 V |
| Wi-Fi | 2.4 GHz |
| Main interface | USB |
| Programming | USB |

---

# 4. Electrical Conductivity Sensor

## 4.1 DFRobot SEN0707

The SEN0707 is the primary conductivity sensor for V0.1.

### Specifications

| Parameter | Specification |
|---|---|
| Sensor | Industrial water conductivity sensor |
| Model | SEN0707 |
| Cell constant | K=10 |
| Measurement range | 10 to 20,000 µS/cm |
| Resolution | 1 µS/cm |
| Accuracy | ±1% FS |
| Interface | RS485 |
| Protocol | Modbus-RTU |
| Supply | 10 to 30 V DC |
| Power | Approximately 0.4 W |
| Protection | IP68 |
| Temperature compensation | Built-in |

The sensor includes calibration solution for initial verification.

### V0.1 connection

```text
ESP32-S3 UART
      │
      ▼
 MAX3485
 RS485 Transceiver
      │
      ▼
  SEN0707
```

The SEN0707 must be connected through a removable external connector.

---

# 5. RS485 Interface

## 5.1 MAX3485

The MAX3485 provides the physical RS485 interface between the ESP32-S3 and the SEN0707.

### Signals

```text
ESP32-S3                 MAX3485                 SEN0707
─────────                ───────                 ───────
UART TX  ──────────────► DI
UART RX  ◄────────────── RO
GPIO     ──────────────► DE/RE
                         A ───────────────────── A
                         B ───────────────────── B
GND      ─────────────── GND ─────────────────── GND
```

### Design requirements

- Use 3.3 V-compatible RS485 hardware.
- Keep RS485 wiring short where practical.
- Use twisted-pair wiring for A/B.
- Keep RS485 wiring away from noisy power switching.
- Provide a removable sensor connector.
- Add termination only when required by the physical bus configuration.
- Avoid unnecessary external pull-ups or pull-downs.
- Protect the ESP32 UART interface from incorrect external wiring.

---

# 6. Ultrasonic Sensor

## 6.1 HC-SR04

The HC-SR04 is selected for V0.1 as a low-cost ultrasonic feasibility sensor.

It is not considered the final production ultrasonic sensor.

### Purpose

The HC-SR04 allows the project to validate:

- Trigger generation.
- Echo timing.
- TOF measurement.
- Distance calculation.
- Measurement sequencing.
- Data logging.
- UI presentation.

### Important limitation

The HC-SR04 is an air ultrasonic ranging module.

V0.1 must not treat it as a laboratory-grade submerged ultrasonic measurement system.

The final acoustic transducer and receiver architecture will be selected after the measurement method has been validated.

---

## 6.2 HC-SR04 Interface

```text
ESP32-S3
   │
   ├──── TRIG ───────────────► HC-SR04 TRIG
   │
   └──── ECHO ◄──── Level Shift ◄──── HC-SR04 ECHO
```

The HC-SR04 ECHO output can be approximately 5 V.

The ESP32-S3 GPIO is 3.3 V logic.

Therefore, the ECHO signal must pass through a suitable voltage divider or level-shifting circuit before reaching the ESP32-S3.

### Example divider

```text
HC-SR04 ECHO
     │
     R1
     │
     ├────────────► ESP32 GPIO
     │
     R2
     │
    GND
```

The resistor values must produce a safe ESP32 input voltage.

---

# 7. TOF Measurement Hardware

The hardware must allow the HC-SR04 to be replaced later.

The ultrasonic sensor must therefore use a removable connector.

### Recommended architecture

```text
ESP32-S3
   │
   ▼
TOF Interface
   │
   ▼
Removable Connector
   │
   ▼
Ultrasonic Sensor
```

The connector should expose only the required signals:

- VCC.
- GND.
- TRIG.
- ECHO.

The firmware must not depend directly on the HC-SR04 implementation.

---

# 8. Display

## 8.1 4-inch ST7796 TFT

The V0.1 display is a 4-inch 480x320 SPI TFT using the ST7796 controller.

### Requirements

- 480x320 resolution.
- SPI interface.
- Non-touch.
- Removable connector.
- Suitable for local outdoor or laboratory prototype viewing.
- 3.3 V logic compatibility.

### Main display

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

The display must receive measurement data from the application layer.

It must not directly access sensor drivers.

---

# 9. Physical Controls

The analyzer uses physical controls instead of a touchscreen.

## 9.1 Rotary Encoder

The rotary encoder provides:

- Menu navigation.
- Value adjustment.
- Selection.
- Push-to-select.

### Signals

```text
Encoder A
Encoder B
Encoder Push
GND
```

GPIO inputs should use suitable pull-up or pull-down configuration.

---

## 9.2 START Button

The START button begins a measurement cycle.

```text
START
  │
  ▼
Measurement Manager
  │
  ▼
Measurement Sequence
```

---

## 9.3 BACK Button

The BACK button:

- Returns to the previous screen.
- Cancels an operation where supported.
- Exits configuration screens.

---

# 10. Real-Time Clock

## 10.1 DS3231

The DS3231 provides timestamps without requiring an internet connection.

### Interface

```text
ESP32-S3
   │
   ├── SDA
   ├── SCL
   └── GND
        │
        ▼
     DS3231
```

I2C is acceptable here because the RTC is an internal board-level device.

The design does not use I2C for external sensor communication.

### Purpose

The RTC is used for:

- Measurement timestamps.
- CSV filenames.
- Event logs.
- System status.
- Measurement history.

---

# 11. MicroSD Storage

## 11.1 MicroSD Module

The system uses MicroSD storage through SPI.

### Purpose

- Measurement logging.
- CSV files.
- System logs.
- Configuration backups where required.

### Example structure

```text
/ECTOF/
    config/
    data/
        2026/
            10/
                08.csv
    logs/
```

### Measurement format

```csv
timestamp,conductivity,tof_us,distance_mm,status
2026-10-08T15:32:10,4820,12.482,18.73,VALID
```

The SD card must be removable.

SD failure must not cause the measurement application to crash.

---

# 12. Battery

## 12.1 Battery Type

V0.1 uses a rechargeable single-cell lithium battery.

Target capacity:

```text
3.7 V
~5000 mAh
```

The actual battery selection must include suitable protection.

---

## 12.2 Battery Protection

The battery system should include:

- Overcharge protection.
- Over-discharge protection.
- Over-current protection.
- Short-circuit protection.

A protected battery or suitable protection circuit should be used.

---

# 13. USB-C Charging

The analyzer requires an integrated USB-C charging solution for the single-cell battery.

### Basic architecture

```text
USB-C
  │
  ▼
Li-ion Charger
  │
  ▼
3.7 V Battery
  │
  ▼
Power Distribution
```

Charging circuitry must be suitable for the selected battery chemistry and capacity.

The enclosure should expose the USB-C connector.

---

# 14. Power Rails

The system requires multiple voltage rails.

```text
3.7 V Battery
      │
      ├──────────────► 3.7 V Battery Rail
      │
      ├── Boost ─────► 12 V Rail
      │                  │
      │                  └── SEN0707
      │
      └── Regulator ──► 5 V Rail
                         │
                         ├── HC-SR04
                         └── TFT if required
                         
3.3 V Regulator
      │
      ├── ESP32-S3
      ├── MAX3485
      ├── DS3231
      ├── MicroSD logic
      └── Other 3.3 V peripherals
```

The actual regulator topology must be finalized based on the selected display, SD module, and power requirements.

---

# 15. SEN0707 12 V Supply

The SEN0707 requires 10 to 30 V DC.

A dedicated boost converter is therefore required.

### Architecture

```text
3.7 V Battery
      │
      ▼
12 V Boost Converter
      │
      ▼
SEN0707
```

The boost converter must provide enough output power for the SEN0707 and maintain a stable voltage during measurement.

The boost converter should be physically separated from sensitive measurement wiring where practical.

---

# 16. Battery Voltage Measurement

V0.1 requires a simple battery indicator.

It does not require battery current measurement.

### Architecture

```text
3.7 V Battery
      │
      ▼
Voltage Divider
      │
      ▼
ESP32-S3 ADC
      │
      ▼
Battery Voltage
      │
      ▼
Estimated Battery %
      │
      ▼
TFT / Web UI
```

The voltage divider must scale the maximum battery voltage to a safe ESP32-S3 ADC input range.

The ADC input must include appropriate protection.

---

# 17. Battery Indicator

The battery indicator is intentionally simple.

It is an estimated battery level, not a precision fuel gauge.

### Example levels

| Battery Voltage | Display |
|---:|---:|
| ≥ 4.10 V | 100% |
| 4.00–4.09 V | 80% |
| 3.90–3.99 V | 60% |
| 3.80–3.89 V | 50% |
| 3.70–3.79 V | 35% |
| 3.60–3.69 V | 20% |
| 3.50–3.59 V | 10% |
| ≤ 3.40 V | Critical |

The actual percentage mapping can be refined during battery testing.

### Low-battery behavior

The system should:

- Display a low-battery warning.
- Display the battery percentage.
- Prevent measurement when battery voltage is below the configured critical threshold if required.
- Continue safe shutdown behavior when the battery is critically low.

---

# 18. No Current or Power Measurement

V0.1 does not include:

- Battery current measurement.
- Battery power measurement.
- INA226.
- Dedicated power monitor.
- Fuel gauge.
- Detailed battery power analytics.

The system only measures battery voltage for a simple battery indicator.

---

# 19. Grounding and Noise Control

The EC sensor and RS485 interface can be affected by electrical noise.

The hardware should therefore separate noisy power paths from measurement communication.

### Guidelines

- Keep boost converter wiring short.
- Keep switching power wiring away from RS485 A/B.
- Use twisted pair for RS485.
- Keep sensor cables away from high-current battery wiring.
- Use a common ground reference where required by the interface.
- Avoid unnecessary ground loops.
- Place decoupling capacitors near active devices.
- Use bulk capacitance near voltage regulators.
- Keep digital switching signals away from sensitive sensor wiring.
- Use shielded sensor cables when practical.

---

# 20. Connector Strategy

External sensors must be removable.

## 20.1 EC Sensor

Use a locking industrial connector where practical.

Target:

```text
M12 connector
```

Signals:

- Power.
- GND.
- RS485 A.
- RS485 B.

---

## 20.2 Ultrasonic Sensor

Use a removable connector.

Signals:

- VCC.
- GND.
- TRIG.
- ECHO.

---

## 20.3 Display

Use a removable internal connector.

Signals include:

- SPI.
- Chip select.
- DC.
- Reset.
- Backlight control if required.
- Power.
- Ground.

---

# 21. GPIO Protection

ESP32-S3 GPIOs must be protected from external voltage exposure.

The design should include:

- Voltage dividers where required.
- Level shifters where required.
- Series resistors where useful.
- Pull-up or pull-down resistors.
- Proper connector pinout.
- ESD protection for externally accessible connections where practical.

The HC-SR04 ECHO input is specifically required to have voltage protection.

---

# 22. Suggested Interface Allocation

The final GPIO numbers must be validated against the selected ESP32-S3 DevKitC-1-N8R8 board and peripherals.

A proposed allocation is:

| Function | Interface |
|---|---|
| SEN0707 TX/RX | UART |
| RS485 DE/RE | GPIO |
| HC-SR04 TRIG | GPIO |
| HC-SR04 ECHO | GPIO |
| TFT | SPI |
| MicroSD | SPI |
| DS3231 | I2C |
| Rotary A | GPIO |
| Rotary B | GPIO |
| Rotary Push | GPIO |
| START | GPIO |
| BACK | GPIO |
| Battery ADC | ADC |
| Wi-Fi | Internal |
| USB | Native USB |

SPI devices may share the SPI bus when electrical and software requirements permit.

Each SPI device must have its own chip-select signal.

---

# 23. Enclosure

The V0.1 enclosure should provide:

- Protection for the electronics.
- Access to the TFT.
- Access to rotary encoder.
- START button.
- BACK button.
- USB-C charging port.
- Power switch.
- EC sensor connector.
- Ultrasonic sensor connector.
- SD card access where practical.
- Ventilation for heat-producing components if required.

The enclosure should keep the battery isolated from heat-producing components.

The SEN0707 connector should be positioned so the external sensor cable does not interfere with user controls.

---

# 24. Physical Layout

A recommended layout is:

```text
┌─────────────────────────────────────────┐
│                                         │
│          4" TFT DISPLAY                 │
│                                         │
│                                         │
├─────────────────────────────────────────┤
│  BACK       ENCODER        START        │
│                                         │
├─────────────────────────────────────────┤
│                                         │
│        MAIN ELECTRONICS                 │
│                                         │
│  ESP32-S3                                │
│  RS485        SD        RTC              │
│                                         │
│        POWER / REGULATORS                │
│                                         │
│        BATTERY                           │
│                                         │
├─────────────────────────────────────────┤
│ USB-C     POWER       EC      ULTRASONIC│
└─────────────────────────────────────────┘
```

The exact enclosure dimensions will be determined after selecting the final modules and battery.

---

# 25. Power and Signal Separation

The physical layout should separate:

### Noisy section

- Boost converter.
- Switching regulators.
- Battery power wiring.
- Backlight power.

### Measurement/control section

- ESP32-S3.
- MAX3485.
- RTC.
- Sensor connectors.
- ADC battery input.

RS485 wiring should be routed away from the boost converter and high-current power paths.

---

# 26. Thermal Considerations

Potential heat sources include:

- 12 V boost converter.
- 5 V regulator.
- 3.3 V regulator.
- TFT backlight.
- ESP32-S3 during Wi-Fi operation.

The enclosure must allow adequate thermal dissipation.

The battery should not be placed directly against high-temperature components.

---

# 27. Hardware Startup Sequence

At power-up:

```text
Power ON
   │
   ▼
ESP32-S3 Boot
   │
   ▼
Initialize GPIO
   │
   ▼
Initialize Power ADC
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
Initialize UART / RS485
   │
   ▼
Initialize EC Sensor
   │
   ▼
Initialize TOF
   │
   ▼
Initialize User Input
   │
   ▼
Start Wi-Fi
   │
   ▼
System READY
```

A failure in one non-critical peripheral must not prevent the rest of the system from starting unless the peripheral is required for safe operation.

---

# 28. Measurement Hardware Sequence

A measurement cycle follows:

```text
START
  │
  ▼
Trigger Ultrasonic Sensor
  │
  ▼
Measure Echo Time
  │
  ▼
Calculate TOF
  │
  ▼
Calculate Distance
  │
  ▼
Read SEN0707
  │
  ▼
Validate Measurement
  │
  ▼
Display Result
  │
  ▼
Log Result
  │
  ▼
READY
```

The exact sequence may be adjusted after prototype testing.

---

# 29. Hardware Fault Conditions

The firmware must detect hardware failures where possible.

### EC sensor

- Sensor disconnected.
- RS485 timeout.
- CRC failure.
- Invalid Modbus response.
- UART error.
- Invalid conductivity value.

### Ultrasonic

- No echo.
- Timeout.
- Out-of-range TOF.
- Invalid distance.
- Sensor disconnected.

### SD card

- Card not detected.
- Mount failure.
- File creation failure.
- Write failure.

### RTC

- RTC not detected.
- Invalid time.
- RTC communication failure.

### Display

- Initialization failure.
- SPI communication failure.

### Battery

- Low voltage.
- Critical voltage.
- ADC failure.

---

# 30. Hardware Safety

The prototype must include:

- Battery protection.
- Proper charger.
- Fuse or resettable fuse where appropriate.
- Protected external connectors.
- Proper insulation.
- Correct polarity protection where appropriate.
- Secure battery mounting.
- No exposed battery terminals.
- No exposed high-voltage circuitry.

The 12 V SEN0707 supply is low voltage but must still be properly insulated and protected.

---

# 31. V0.1 Hardware BOM

| Category | Component | Purpose |
|---|---|---|
| MCU | ESP32-S3-DevKitC-1-N8R8 | Main controller |
| EC | DFRobot SEN0707 | Conductivity measurement |
| RS485 | MAX3485 module/transceiver | SEN0707 communication |
| TOF | HC-SR04 | Ultrasonic prototype |
| Display | 4" 480x320 ST7796 TFT | Local UI |
| Storage | MicroSD SPI module | Data logging |
| Storage | 16/32 GB MicroSD | Measurement storage |
| RTC | DS3231 | Timestamping |
| Input | Rotary encoder | Navigation |
| Input | START button | Measurement trigger |
| Input | BACK button | Navigation |
| Battery | 3.7 V ~5000 mAh Li-ion | Portable power |
| Charger | USB-C 1S Li-ion charger | Battery charging |
| Boost | 3.7 V to 12 V | SEN0707 power |
| Regulator | 3.7 V to 5 V | 5 V peripherals |
| ADC | ESP32-S3 internal ADC | Battery voltage |
| Connector | M12 | EC sensor |
| Connector | Removable connector | Ultrasonic sensor |
| Protection | Fuse/polyfuse | Power protection |
| Enclosure | Custom project enclosure | Mechanical protection |

---

# 32. V0.1 Hardware Exclusions

The following are intentionally excluded from the V0.1 hardware:

- Production-grade ultrasonic transducer.
- Dedicated ultrasonic receiver amplifier.
- Precision TOF acquisition hardware.
- Battery current measurement.
- Battery power measurement.
- INA226.
- Dedicated fuel gauge.
- Advanced battery monitoring.
- Touchscreen.
- Cellular communication.
- GPS.
- Cloud connectivity.
- Industrial EMC certification.
- Production enclosure certification.

These may be evaluated after the V0.1 prototype validates the measurement concept.

---

# 33. Hardware Upgrade Path

The hardware must allow future upgrades without requiring a complete redesign.

### Possible V0.2 upgrades

- Laboratory-grade ultrasonic transducer.
- Dedicated ultrasonic receiver.
- Higher-precision TOF timing hardware.
- Improved acoustic coupling.
- Improved sample fixture.
- Better EC sample cell.
- More robust sensor connectors.
- Improved power management.
- Larger display.
- Better enclosure.
- Additional calibration features.

The ESP32-S3 application architecture should remain unchanged where possible.

---

# 34. Hardware Design Acceptance Criteria

The V0.1 hardware is considered ready for firmware integration when:

- ESP32-S3 boots reliably.
- TFT operates correctly.
- Rotary encoder works.
- START button works.
- BACK button works.
- DS3231 provides valid time.
- MicroSD mounts reliably.
- SEN0707 communicates through RS485.
- HC-SR04 produces valid prototype measurements.
- HC-SR04 ECHO is safely level-shifted.
- Battery voltage can be measured safely.
- Battery indicator can be displayed.
- USB-C charging operates correctly.
- 12 V SEN0707 supply is stable.
- All external sensors are removable.
- Wiring is mechanically secure.
- No exposed unsafe electrical connections exist.
- The complete system can operate from the battery.

---

# 35. Final V0.1 Hardware Architecture

The finalized V0.1 hardware is:

```text
                         ┌───────────────────────┐
                         │       ESP32-S3        │
                         │   DevKitC-1-N8R8      │
                         └───────────┬───────────┘
                                     │
        ┌────────────────────────────┼───────────────────────────┐
        │                            │                           │
        ▼                            ▼                           ▼
   UART / RS485                 SPI Display                  GPIO
        │                            │                           │
        ▼                            ▼                           ├── START
   MAX3485                       ST7796 TFT                     ├── BACK
        │                                                        ├── Encoder
        ▼                                                        │
    SEN0707                                                       │
                                                                  │
        ┌────────────────────────────┼───────────────────────────┤
        │                            │                           │
        ▼                            ▼                           ▼
    TOF GPIO                     SPI MicroSD                  I2C RTC
        │                            │                           │
        ▼                            ▼                           ▼
    HC-SR04                      Data Storage                  DS3231
        │
        │
   ECHO Level Shift
        │
        ▼
    ESP32 GPIO


                    POWER SYSTEM

             ┌───────────────────────┐
             │   3.7 V Li-ion        │
             │   ~5000 mAh           │
             └───────────┬───────────┘
                         │
             ┌───────────┼────────────┐
             │           │            │
             ▼           ▼            ▼
          12 V Boost    5 V Rail    3.3 V Rail
             │           │            │
             ▼           ▼            ▼
          SEN0707     TFT / TOF    ESP32 + Logic

                         │
                         ▼
                    Voltage Divider
                         │
                         ▼
                    ESP32 ADC
                         │
                         ▼
                  Battery Percentage
```

This architecture is the hardware baseline for EC-TOF Analyzer V0.1.