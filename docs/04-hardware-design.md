# EC-TOF Analyzer

## Hardware Design

Version: 0.1  
Status: Prototype Planning  
Related Documents:

- `01-product-requirements.md`
- `02-technical-requirements.md`
- `03-system-architecture.md`

---

# 1. Purpose

This document defines the hardware design for the EC-TOF Analyzer V0.1 prototype.

It covers:

- Hardware components
- Controller
- Sensor interfaces
- GPIO allocation
- Power architecture
- RS485 interface
- Ultrasonic interface
- Display interface
- SD card interface
- RTC interface
- User controls
- Battery and charging
- Connectors
- Electrical protection
- Wiring
- Enclosure layout
- Bill of materials
- Hardware expansion points

The design prioritizes readily available modules and components.

A custom PCB is not required for V0.1.

---

# 2. Hardware Design Principles

The hardware shall follow these principles:

1. Use off-the-shelf components.
2. Keep sensors removable.
3. Keep the ESP32-S3 development board replaceable.
4. Avoid I2C for external sensors.
5. Keep noisy power circuits separated from sensitive signal wiring.
6. Protect ESP32-S3 GPIOs from incompatible voltage levels.
7. Use connectors instead of permanent sensor wiring.
8. Provide accessible test points during development.
9. Use modular power converters during V0.1.
10. Leave expansion capability for future hardware.

---

# 3. Hardware Block Diagram

```text
                           EC-TOF ANALYZER
                                  │
                         ┌────────▼────────┐
                         │   ESP32-S3      │
                         │ DevKitC-1-N8R8  │
                         └────────┬────────┘
                                  │
         ┌────────────────────────┼────────────────────────┐
         │                        │                        │
         ▼                        ▼                        ▼
    EC Interface             TOF Interface             Display
         │                        │                        │
      UART                    GPIO/RMT                  SPI
         │                        │                        │
      MAX3485                 HC-SR04                 ST7796
         │
      RS485
         │
     SEN0707

         ┌────────────────────────┼────────────────────────┐
         │                        │                        │
         ▼                        ▼                        ▼
       RTC                       SD                    Controls
         │                        │                        │
       I2C                       SPI                      GPIO
         │                        │                        │
     DS3231                  MicroSD Card          Encoder/Buttons

                                  │
                                  ▼
                             Power System
                                  │
                           ┌──────┴──────┐
                           │             │
                        Battery       USB-C
```

---

# 4. Main Controller

## 4.1 ESP32-S3 Development Board

Selected controller:

**ESP32-S3-DevKitC-1-N8R8**

Responsibilities:

- Main application processor
- Wi-Fi
- Sensor communication
- Display control
- User input
- SD card
- RTC
- Measurement processing
- Data logging
- Web server

The development board shall remain removable during the prototype stage.

---

# 5. Component Selection

## 5.1 Primary Components

| Component | Selection | Purpose |
|---|---|---|
| MCU | ESP32-S3-DevKitC-1-N8R8 | Main controller |
| EC Sensor | DFRobot SEN0707 | Conductivity measurement |
| RS485 | MAX3485 module | EC communication |
| Ultrasonic | HC-SR04 | V0.1 TOF prototype |
| Display | 4" 480×320 ST7796 SPI TFT | Local UI |
| RTC | DS3231 | Date/time |
| Storage | MicroSD SPI module | Data logging |
| Battery | 3.7 V 5000 mAh Li-ion/LiPo | Portable power |
| Charger | USB-C 1S Li-ion charger | Battery charging |
| EC Boost | 3.7 V → 12 V boost | SEN0707 power |
| Peripheral Supply | 3.7 V → 5 V converter | TFT/HC-SR04 |
| Input | Rotary encoder | Navigation |
| Input | START button | Measurement |
| Input | BACK button | Navigation |
| Main Switch | SPST power switch | Power control |
| Protection | Fuse/polyfuse | Power protection |

---

# 6. Power Architecture

The battery is the primary power source.

```text
                    3.7 V Battery
                          │
                          ▼
                    Main Power Switch
                          │
             ┌────────────┼─────────────┐
             │            │             │
             ▼            ▼             ▼
          12 V Boost    5 V Rail     ESP32 Power
             │            │
             ▼       ┌────┴────┐
          SEN0707     │         │
                      ▼         ▼
                     TFT      HC-SR04
```

The actual 3.3 V supply for logic shall follow the selected ESP32-S3 development board's recommended power input.

---

# 7. Voltage Domains

The prototype shall use three primary voltage domains.

## 7.1 Battery Domain

Nominal:

```text
3.7 V
```

Actual lithium battery voltage varies during charging and discharge.

Expected operating range depends on the selected battery and protection circuit.

---

## 7.2 12 V Sensor Domain

Used by:

```text
SEN0707
```

Target:

```text
12 V DC
```

The boost converter shall provide sufficient current for the SEN0707 with engineering margin.

The exact converter shall be selected based on measured system load.

---

## 7.3 5 V Peripheral Domain

Used by:

```text
HC-SR04
TFT
```

The exact TFT supply requirement must be verified against the selected module.

If the TFT module requires 3.3 V instead of 5 V, the module shall be powered according to its specific electrical design.

---

## 7.4 3.3 V Logic Domain

Used by:

```text
ESP32-S3
MAX3485
DS3231
Input logic
```

All signals connected directly to ESP32-S3 GPIOs must remain within ESP32-S3 voltage limits.

---

# 8. Battery

## 8.1 Battery Specification

Initial target:

```text
Type: Rechargeable Li-ion / LiPo
Nominal voltage: 3.7 V
Capacity: 5000 mAh
Cells: 1S
```

The battery shall include suitable protection.

A protected battery is preferred for the prototype.

---

# 9. Battery Charging

The battery shall be charged through USB-C.

Basic architecture:

```text
USB-C
   │
   ▼
1S Li-ion Charger
   │
   ▼
3.7 V Battery
   │
   ▼
System Power
```

The charger shall provide:

- Overcharge protection
- Over-discharge protection
- Over-current protection
- Short-circuit protection where supported

The charger must be compatible with the selected battery chemistry.

---

# 10. Battery Protection

The system shall include protection against:

- Overcharge
- Over-discharge
- Short circuit
- Excessive current

Battery protection should preferably be implemented by a dedicated protection circuit or protected battery.

The ESP32 firmware shall not be responsible for primary battery safety.

---

# 11. Battery Monitoring

Battery monitoring shall be implemented separately from the main power path.

Possible V0.1 implementation:

```text
Battery
   │
   ▼
Voltage Divider
   │
   ▼
ESP32-S3 ADC
```

A dedicated fuel gauge may be added later.

Battery percentage shall be treated as an estimate when calculated only from battery voltage.

---

# 12. 12 V Boost Converter

The SEN0707 requires a higher voltage supply than the battery provides.

Architecture:

```text
3.7 V Battery
      │
      ▼
12 V Boost Converter
      │
      ▼
SEN0707
```

The converter shall:

- Accept the battery voltage range.
- Produce a stable 12 V output.
- Provide sufficient current.
- Include appropriate input/output capacitors.
- Be physically separated from sensitive signal wiring where practical.

The converter shall be enabled only when required if power testing shows that this significantly improves battery life.

---

# 13. 5 V Converter

A separate 5 V regulator or boost converter shall be used where required.

```text
3.7 V Battery
      │
      ▼
5 V Converter
      │
      ├── TFT
      └── HC-SR04
```

The actual TFT supply voltage must be confirmed before final wiring.

---

# 14. Main Power Switch

A physical power switch shall be installed on the enclosure.

Recommended location:

```text
Rear / Side Panel
```

The switch shall disconnect system power from the battery.

The charging input should remain available according to the selected charger/power-path design.

---

# 15. Power Protection

The main battery output should include a fuse or resettable polyfuse.

Recommended architecture:

```text
Battery
   │
   ▼
Fuse / Polyfuse
   │
   ▼
Main Switch
   │
   ▼
Power Distribution
```

The fuse rating shall be selected after measuring the expected maximum system current.

---

# 16. Conductivity Sensor

Selected sensor:

**DFRobot SEN0707**

The sensor shall be externally mounted and removable.

The sensor connection shall include:

- Power
- RS485 A
- RS485 B
- Ground

The sensor shall receive power from the dedicated 12 V rail.

---

# 17. EC Sensor Connection

Architecture:

```text
             SEN0707
          ┌───────────┐
          │           │
  +12 V ──┤ Power     │
   GND ───┤ Ground    │
    A ────┤ RS485 A   │
    B ────┤ RS485 B   │
          └───────────┘
                │
                │
             M12 Cable
                │
                ▼
          Panel Connector
                │
                ▼
             MAX3485
                │
                ▼
            ESP32-S3
```

The final M12 pin assignment shall be verified against the selected connector and SEN0707 cable configuration before assembly.

---

# 18. RS485 Interface

The MAX3485 shall provide the physical RS485 interface.

```text
ESP32-S3
    │
 UART
    │
    ▼
MAX3485
    │
    ├── RO → ESP32 RX
    ├── DI ← ESP32 TX
    ├── DE ← ESP32 GPIO
    └── RE ← ESP32 GPIO
    │
    ├── A
    └── B
```

Depending on the selected module, DE and RE may be combined.

The final circuit shall follow the actual MAX3485 module design.

---

# 19. RS485 Wiring

Use twisted-pair wiring for:

```text
RS485 A
RS485 B
```

Recommended:

```text
Pair 1:
A + B
```

Power and ground should use separate conductors.

For longer cables, shielded cable is preferred.

---

# 20. RS485 Termination

A 120 Ω termination resistor may be required depending on cable length and topology.

For a single short sensor connection, termination requirements should be evaluated during testing.

The prototype should provide an accessible location for adding or removing termination.

---

# 21. Ultrasonic Sensor

Selected V0.1 sensor:

**HC-SR04**

Connection:

```text
HC-SR04
├── VCC
├── GND
├── TRIG
└── ECHO
```

The sensor shall be connected through a removable connector.

---

# 22. HC-SR04 Voltage Protection

The HC-SR04 ECHO signal may be 5 V.

The ESP32-S3 GPIO shall not receive the raw signal.

Use:

```text
HC-SR04 ECHO
      │
      ▼
Voltage Divider
      │
      ▼
ESP32-S3 ECHO GPIO
```

Example divider concept:

```text
ECHO
  │
  R1
  │
  ├────────── ESP32 GPIO
  │
  R2
  │
 GND
```

The resistor values shall be selected to keep the GPIO input safely within the ESP32-S3 voltage range.

The exact values will be finalized during schematic implementation.

---

# 23. Ultrasonic Trigger

The ESP32-S3 shall drive the HC-SR04 TRIG input.

The trigger output must remain within the HC-SR04 input voltage requirements.

Architecture:

```text
ESP32 GPIO
    │
    ▼
HC-SR04 TRIG
```

A small series resistor may be added for signal integrity if required.

---

# 24. Ultrasonic Connector

The HC-SR04 shall use a removable connector.

Minimum signals:

```text
VCC
GND
TRIG
ECHO
```

The connector should prevent accidental reversal where practical.

---

# 25. Display

Selected display:

```text
4-inch
480 × 320
ST7796
SPI
```

The display shall be mounted on the front panel.

The display connection should be removable.

---

# 26. TFT Interface

Typical signals:

```text
SCLK
MOSI
MISO
CS
DC
RESET
BACKLIGHT
```

The actual signals required depend on the selected ST7796 module.

The display module's schematic shall be checked before final wiring.

---

# 27. Display Power

The TFT shall be powered according to the selected module's requirements.

The display supply must not be assumed solely from the ST7796 controller voltage.

The final module must be verified for:

- Logic voltage
- Backlight voltage
- Input voltage
- Current consumption

---

# 28. MicroSD

The system shall use a MicroSD card for measurement storage.

Connection:

```text
ESP32-S3
   │
   │ SPI
   ├── SCLK
   ├── MOSI
   ├── MISO
   └── CS
        │
        ▼
    MicroSD Module
        │
        ▼
      SD Card
```

The SD card should be removable.

---

# 29. SD Card Requirements

Recommended initial card:

```text
16 GB or 32 GB
```

A reputable card should be used for testing.

The firmware shall format and use a standard filesystem supported by ESP-IDF.

The card should be formatted before first use.

---

# 30. RTC

Selected RTC:

**DS3231**

Connection:

```text
ESP32-S3
    │
    │ I2C
    ├── SDA
    └── SCL
         │
         ▼
       DS3231
```

The RTC shall have its backup battery installed.

---

# 31. User Controls

The front panel shall contain:

```text
┌──────────────────────────────┐
│                              │
│        TFT DISPLAY           │
│                              │
│                              │
└──────────────────────────────┘

       [ ROTARY ENCODER ]

    [ START ]       [ BACK ]
```

The physical layout may be changed after enclosure prototyping.

---

# 32. Rotary Encoder

The encoder shall provide:

```text
A
B
SW
VCC
GND
```

GPIO inputs shall use internal or external pull-up resistors as appropriate.

Software debounce shall be implemented.

---

# 33. START Button

The START button shall use a digital GPIO.

Recommended connection:

```text
GPIO
 │
 ├── Internal Pull-up
 │
 └── Button
       │
      GND
```

Pressed state:

```text
LOW
```

Released state:

```text
HIGH
```

---

# 34. BACK Button

The BACK button shall use the same basic electrical arrangement.

```text
GPIO
 │
 ├── Internal Pull-up
 │
 └── Button
       │
      GND
```

---

# 35. GPIO Allocation

The following is the proposed V0.1 GPIO map.

| Function | GPIO | Interface |
|---|---:|---|
| EC UART TX | GPIO17 | UART |
| EC UART RX | GPIO18 | UART |
| RS485 DE/RE | GPIO16 | GPIO |
| Ultrasonic TRIG | GPIO4 | GPIO |
| Ultrasonic ECHO | GPIO5 | GPIO/RMT |
| TFT SCLK | GPIO12 | SPI |
| TFT MOSI | GPIO11 | SPI |
| TFT MISO | GPIO13 | SPI |
| TFT CS | GPIO10 | GPIO |
| TFT DC | GPIO9 | GPIO |
| TFT RESET | GPIO8 | GPIO |
| TFT Backlight | GPIO7 | GPIO/PWM |
| SD SCLK | GPIO36 | SPI |
| SD MOSI | GPIO35 | SPI |
| SD MISO | GPIO37 | SPI |
| SD CS | GPIO34 | GPIO |
| RTC SDA | GPIO6 | I2C |
| RTC SCL | GPIO15 | I2C |
| Encoder A | GPIO1 | GPIO |
| Encoder B | GPIO2 | GPIO |
| Encoder SW | GPIO3 | GPIO |
| START | GPIO38 | GPIO |
| BACK | GPIO39 | GPIO |
| Battery ADC | GPIO14 | ADC |

This GPIO map is a starting allocation and must be validated against the actual ESP32-S3-DevKitC-1-N8R8 pin availability and the selected display/module wiring before hardware assembly.

Reserved or boot-sensitive pins shall be avoided where practical.

---

# 36. GPIO Design Rules

The following rules apply:

- Do not connect 5 V signals directly to ESP32-S3 GPIOs.
- Avoid boot-strapping pins where possible.
- Avoid using flash/PSRAM-connected pins.
- Keep high-speed signals short.
- Use pull-ups or pull-downs where required.
- Add series resistors if signal integrity requires them.
- Keep sensor signals away from switching converter nodes.

---

# 37. Proposed Interface Summary

```text
ESP32-S3
│
├── UART
│    └── MAX3485 → SEN0707
│
├── SPI Bus 1
│    └── ST7796 TFT
│
├── SPI Bus 2
│    └── MicroSD
│
├── I2C
│    └── DS3231
│
├── GPIO
│    ├── HC-SR04 TRIG
│    ├── HC-SR04 ECHO
│    ├── Encoder A
│    ├── Encoder B
│    ├── Encoder SW
│    ├── START
│    └── BACK
│
└── ADC
     └── Battery Voltage
```

---

# 38. Recommended SPI Strategy

The prototype should preferably use separate SPI hosts for:

```text
SPI Display
SPI MicroSD
```

This reduces:

- Bus contention
- Driver complexity
- Display/SD timing conflicts

If ESP32-S3 SPI resources or GPIO routing make this impractical, both devices may share a bus with independent CS lines.

---

# 39. Grounding Strategy

The system shall use a common system ground unless a future design introduces galvanic isolation.

Ground paths should be arranged so that high-current converter return currents do not unnecessarily pass through sensitive signal paths.

Conceptually:

```text
Battery GND
    │
    ├── Power Converter GND
    │
    ├── ESP32 GND
    │
    ├── RS485 GND
    │
    ├── TFT GND
    │
    ├── SD GND
    │
    └── Sensor GND
```

The physical wiring should use a controlled star or low-impedance distribution approach where practical.

---

# 40. Noise Reduction

The following should be implemented:

- 100 nF local bypass capacitors near digital modules where required.
- Bulk capacitance near DC-DC converters.
- Short power paths.
- Twisted RS485 pair.
- Physical separation between boost converter and sensor wiring.
- Separate routing for switching power and measurement signals.
- Ferrite filtering if testing shows significant noise.
- Shielding for long external sensor cables where practical.

---

# 41. Decoupling

Each major module should have appropriate local decoupling.

Minimum concept:

```text
Power Rail
    │
    ├── Bulk Capacitor
    │
    └── 100 nF Ceramic
             │
           Module
```

Actual capacitor values shall follow the module and regulator requirements.

---

# 42. Test Points

The prototype wiring should expose test points for:

```text
TP1  Battery Voltage
TP2  5 V Rail
TP3  12 V Rail
TP4  3.3 V Logic
TP5  RS485 A
TP6  RS485 B
TP7  Ultrasonic TRIG
TP8  Ultrasonic ECHO
TP9  GND
```

These test points will simplify debugging.

---

# 43. Connector Strategy

Recommended connector groups:

| Connection | Connector |
|---|---|
| EC Sensor | M12 |
| Ultrasonic | JST or locking 4-pin |
| Battery | JST |
| TFT | Locking header/JST |
| SD | Module/socket |
| RTC | JST/header |
| Buttons | JST/header |
| USB-C | Panel-mounted or board connector |

External connectors should be keyed or positioned to reduce incorrect installation.

---

# 44. EC M12 Connector

The M12 connector shall be mounted on the enclosure.

The connector must be compatible with the SEN0707 cable.

The exact pinout must be confirmed before final assembly.

Do not assume the M12 pin numbering from a generic connector is identical to the sensor cable.

---

# 45. Enclosure Layout

Initial enclosure concept:

```text
FRONT

┌────────────────────────────────────────┐
│                                        │
│             4" TFT DISPLAY             │
│                                        │
│                                        │
├────────────────────────────────────────┤
│                                        │
│              ROTARY                     │
│              ENCODER                    │
│                                        │
│       START              BACK           │
│                                        │
└────────────────────────────────────────┘
```

Rear/side:

```text
┌──────────────────────────────┐
│ USB-C                        │
│                              │
│ POWER SWITCH                 │
│                              │
│ EC M12                       │
│                              │
│ ULTRASONIC CONNECTOR         │
└──────────────────────────────┘
```

---

# 46. Internal Layout

Recommended internal arrangement:

```text
┌───────────────────────────────────────┐
│                                       │
│              TFT BACK                 │
│                                       │
├───────────────────────────────────────┤
│                                       │
│ ESP32-S3              SD Module       │
│                                       │
│ MAX3485               DS3231          │
│                                       │
├───────────────────────────────────────┤
│                                       │
│ Power Converters      Battery         │
│                                       │
└───────────────────────────────────────┘
```

The boost converter should be kept away from the RS485 and ultrasonic signal paths.

---

# 47. Thermal Considerations

The following components may generate heat:

- 12 V boost converter
- 5 V converter
- ESP32-S3
- TFT backlight regulator
- Battery during charging

The enclosure should provide sufficient airflow or thermal conduction.

The battery should not be positioned directly against a hot converter.

---

# 48. Battery Placement

The battery should be:

- Mechanically secured
- Protected from sharp edges
- Protected from excessive heat
- Away from high-temperature components
- Replaceable during prototype development

The battery shall not be allowed to move freely inside the enclosure.

---

# 49. Wiring Requirements

Internal wiring shall be organized by function.

Recommended separation:

```text
POWER
├── Battery
├── 12 V
└── 5 V

DIGITAL
├── SPI
├── I2C
└── GPIO

SENSOR
├── RS485
└── Ultrasonic
```

High-current wires should be kept short.

External sensor cables should have strain relief.

---

# 50. Hardware Failure Conditions

The hardware design shall account for:

- Sensor disconnect
- Shorted sensor cable
- Reversed connector where possible
- SD card removal
- Battery undervoltage
- Converter failure
- RS485 wiring fault
- Ultrasonic connector disconnect

The firmware shall detect failures where electrical detection is possible.

---

# 51. Bill of Materials

Initial prototype BOM:

| # | Component | Qty | Purpose |
|---:|---|---:|---|
| 1 | ESP32-S3-DevKitC-1-N8R8 | 1 | Main controller |
| 2 | DFRobot SEN0707 | 1 | EC measurement |
| 3 | MAX3485 RS485 module | 1 | RS485 interface |
| 4 | HC-SR04 | 1 | Ultrasonic prototype |
| 5 | 4" 480×320 ST7796 SPI TFT | 1 | Display |
| 6 | MicroSD SPI module | 1 | SD interface |
| 7 | 16/32 GB MicroSD card | 1 | Data storage |
| 8 | DS3231 RTC module | 1 | RTC |
| 9 | Rotary encoder | 1 | Navigation |
| 10 | START push button | 1 | Measurement control |
| 11 | BACK push button | 1 | Navigation |
| 12 | 3.7 V 5000 mAh battery | 1 | Main power |
| 13 | USB-C 1S Li-ion charger | 1 | Battery charging |
| 14 | 3.7 V → 12 V boost converter | 1 | SEN0707 supply |
| 15 | 3.7 V → 5 V converter | 1 | Peripheral supply |
| 16 | Main power switch | 1 | Power control |
| 17 | Fuse/polyfuse | 1 | Protection |
| 18 | M12 connector | 1 set | EC sensor |
| 19 | 4-pin removable connector | 1 set | Ultrasonic |
| 20 | JST connectors | Several | Internal wiring |
| 21 | Resistors | Several | ECHO level shifting |
| 22 | Capacitors | Several | Decoupling |
| 23 | Prototype wiring | As required | Assembly |
| 24 | Enclosure | 1 | Mechanical housing |

---

# 52. Prototype Assembly Strategy

The prototype should be assembled in stages.

## Stage 1

Controller only:

```text
ESP32-S3
```

Verify:

- Programming
- Boot
- Serial output
- Wi-Fi

## Stage 2

Add display.

```text
ESP32-S3
    ↓
TFT
```

Verify UI.

## Stage 3

Add RTC.

```text
ESP32-S3
    ↓
DS3231
```

Verify timestamp.

## Stage 4

Add SD.

Verify file creation and CSV logging.

## Stage 5

Add RS485.

```text
ESP32-S3
    ↓
MAX3485
    ↓
SEN0707
```

Verify conductivity.

## Stage 6

Add ultrasonic.

```text
ESP32-S3
    ↓
Level Shifter
    ↓
HC-SR04
```

Verify TOF.

## Stage 7

Add battery system.

Verify:

- Startup
- Runtime
- Charging
- Current consumption

## Stage 8

Integrate the complete system.

---

# 53. Hardware Bring-Up Order

Recommended order:

```text
ESP32-S3
   ↓
3.3 V
   ↓
TFT
   ↓
RTC
   ↓
SD
   ↓
RS485
   ↓
SEN0707
   ↓
HC-SR04
   ↓
Buttons
   ↓
Battery
   ↓
Power converters
   ↓
Complete system
```

This prevents multiple unknown variables from being introduced at the same time.

---

# 54. Prototype Wiring Rule

Do not build the complete system on a breadboard if the sensor cables and power converters introduce significant noise.

For early development:

- Breadboard digital logic where convenient.
- Use proper screw terminals/connectors for power.
- Use short wires for SPI.
- Use twisted pair for RS485.
- Use removable connectors for external sensors.
- Move to perfboard or a mounting plate once the design is stable.

---

# 55. Hardware Validation

Before proceeding to enclosure integration, verify:

### Power

- Battery voltage
- 5 V rail
- 12 V rail
- Logic voltage
- Converter temperature
- Current consumption

### EC

- RS485 communication
- Modbus response
- Sensor reading
- Sensor disconnect

### Ultrasonic

- Trigger
- Echo level
- TOF
- Distance
- Timeout

### Display

- Initialization
- Full-screen rendering
- Backlight
- Touch not required

### SD

- Initialization
- File write
- File read
- Card removal

### RTC

- Read
- Write
- Battery backup

### Controls

- Encoder
- Encoder button
- START
- BACK

---

# 56. Hardware Design Risks

| Risk | Impact | Mitigation |
|---|---|---|
| HC-SR04 unsuitable for final measurement | High | Treat as V0.1 feasibility hardware |
| 5 V ECHO damages ESP32 | High | Use level shifting |
| Boost converter introduces noise | High | Physical separation and filtering |
| Battery runtime too short | Medium | Measure actual current and optimize |
| SD bus conflicts with TFT | Medium | Prefer separate SPI hosts |
| RS485 noise | Medium | Twisted pair and proper grounding |
| Incorrect M12 wiring | High | Verify SEN0707 cable pinout |
| TFT voltage mismatch | High | Verify selected module before connection |
| Converter overheating | Medium | Measure thermal performance |
| Battery protection inadequate | High | Use protected battery/charger |

---

# 57. Hardware Expansion Points

The prototype should leave room for:

- Dedicated ultrasonic TX/RX electronics
- Better ultrasonic transducer
- Fuel gauge
- External ADC
- Additional sensors
- USB communication
- Custom PCB
- Isolated RS485
- Improved power management
- Hardware emergency stop if required
- Additional measurement channels

These are not required for V0.1.

---

# 58. Final Hardware Architecture

The final V0.1 hardware architecture is:

```text
                           ┌─────────────────┐
                           │  3.7 V Battery  │
                           └────────┬────────┘
                                    │
                              Fuse / Switch
                                    │
                ┌───────────────────┼──────────────────┐
                │                   │                  │
                ▼                   ▼                  ▼
          12 V Boost             5 V Rail          ESP32-S3
                │                   │                  │
                ▼              ┌────┴────┐             │
             SEN0707            │         │             │
                ▲               ▼         ▼             │
                │              TFT     HC-SR04          │
                │                                        │
                │             ┌──────────────────────────┤
                │             │                          │
                │             ▼                          ▼
                │          MAX3485                    SPI/I2C/GPIO
                │             ▲                          │
                └─────────────┘                          │
                                                       │
                    ┌──────────────────────────────────┼──────────┐
                    │                                  │          │
                    ▼                                  ▼          ▼
                  DS3231                              SD       Controls
                   RTC                               Card
```

---

# 59. Hardware Baseline

The V0.1 hardware baseline is:

| Subsystem | Hardware |
|---|---|
| Controller | ESP32-S3-DevKitC-1-N8R8 |
| EC Sensor | DFRobot SEN0707 |
| EC Communication | MAX3485 + RS485 |
| Ultrasonic | HC-SR04 |
| Display | 4" 480×320 ST7796 SPI TFT |
| RTC | DS3231 |
| Storage | MicroSD |
| User Input | Rotary encoder + START + BACK |
| Battery | 3.7 V 5000 mAh |
| Charger | USB-C 1S charger |
| EC Power | 12 V boost |
| Peripheral Power | 5 V converter |
| Main Protection | Fuse/polyfuse |
| EC Connector | M12 |
| Ultrasonic Connector | Removable 4-pin |
| Enclosure | Portable prototype enclosure |
| PCB | No custom PCB for V0.1 |

---

# 60. Hardware Design Status

The hardware design is considered:

**Prototype Ready for Detailed Wiring and Bench Validation**

Before applying power to the complete system, the following must be verified against the actual modules purchased:

1. ESP32-S3 GPIO availability.
2. ST7796 module voltage requirements.
3. ST7796 module pinout.
4. MicroSD module voltage compatibility.
5. MAX3485 module wiring.
6. SEN0707 cable/M12 pinout.
7. HC-SR04 ECHO voltage.
8. Battery charger/protection design.
9. 12 V boost converter current capability.
10. 5 V converter current capability.
11. Final battery current requirement.

The GPIO table in this document is a proposed allocation, not a final electrical schematic.

The next hardware implementation step should be creating the detailed wiring/schematic from this architecture before physical assembly.