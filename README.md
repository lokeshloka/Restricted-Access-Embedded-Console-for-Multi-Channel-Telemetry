# Restricted Access Embedded Console for Multi-Channel Telemetry

An embedded monitoring and control system that provides **restricted access to multi-channel telemetry data** using keypad-based authentication. After successful authentication, an authorized user can monitor **temperature and voltage** through an LCD interface.

The system uses an **LPC21xx ARM7 microcontroller**, an **LM35DZ/NOPB temperature sensor**, an **MCP3204 ADC**, a potentiometer for voltage monitoring, an LCD, keypad, I²C memory, and a GSM modem connected through UART.

When the temperature exceeds a predefined threshold, the system displays the high-temperature condition on the LCD and sends an alert through **UART to the GSM modem**, which then sends the notification to the user's mobile phone.

---

## Features

- 🔐 Password-protected access using a keypad
- 🔢 Six-digit password authentication
- 💾 Password storage using I²C memory
- 🌡️ Temperature measurement using **LM35DZ/NOPB**
- ⚡ Voltage measurement using a **potentiometer**
- 🔌 MCP3204 ADC with two active analog channels
- 📺 16×2 LCD for local monitoring
- 📡 UART communication with GSM modem
- 📱 Mobile alert when temperature exceeds the threshold
- 🚨 Temperature threshold monitoring
- 💡 LED status indication
- ⚙️ Motor/control output for critical temperature conditions

---

## System Architecture

```text
                         ┌─────────────────────┐
                         │      LPC21xx MCU    │
                         │   Main Controller   │
                         └──────────┬──────────┘
                                    │
             ┌──────────────────────┼──────────────────────┐
             │                      │                      │
             ▼                      ▼                      ▼
        ┌─────────┐             ┌─────────┐           ┌─────────┐
        │ Keypad  │             │   LCD   │           │  UART   │
        │  Input  │             │ Display │           │  GSM    │
        └────┬────┘             └─────────┘           └────┬────┘
             │                                             │
             ▼                                             ▼
        ┌───────────┐                                User Mobile
        │ I²C Memory│
        │ Password  │
        └───────────┘

                                    │
                                    ▼
                             ┌─────────────┐
                             │ MCP3204 ADC │
                             │ SPI         │
                             └──────┬──────┘
                                    │
                    ┌───────────────┴───────────────┐
                    │                               │
                    ▼                               ▼
              ADC Channel 0                   ADC Channel 1
                    │                               │
                    ▼                               ▼
              LM35DZ/NOPB                    Potentiometer
              Temperature                     Voltage Input
                    │                               │
                    ▼                               ▼
              Temperature                       Voltage
              Monitoring                       Monitoring

Multi-Channel Telemetry

The system uses an MCP3204 ADC connected to the LPC21xx through SPI.
SPI/ADC Channel	Input Device	Parameter
Channel 0	LM35DZ/NOPB	Temperature
Channel 1	Potentiometer	Voltage
ADC Channel Mapping

SPI/ADC CH0 → LM35 → Temperature

SPI/ADC CH1 → Potentiometer → Voltage

Temperature Sensor — LM35

The temperature sensor used in the project is:

Texas Instruments LM35DZ/NOPB
Parameter	Specification
Manufacturer	Texas Instruments
Part Number	LM35DZ/NOPB
Package	TO-92
Number of Pins	3
Output	Analog
Temperature coefficient	10 mV/°C
Supply voltage	4 V to 30 V
Temperature range	0°C to 100°C

The LM35 produces an analog voltage proportional to the measured temperature.

VOUT = 10 mV × Temperature (°C)

Example LM35 Output

Temperature       LM35 Output

0°C       →       0.00 V
25°C      →       0.25 V
30°C      →       0.30 V
50°C      →       0.50 V
100°C     →       1.00 V

LM35 Pin Configuration

The LM35 is connected to the ADC input as follows:

              LM35
           ┌─────────┐
 VCC ──────┤ VCC     │
 ADC ◄─────┤ VOUT    │
 GND ──────┤ GND     │
           └─────────┘

LM35 Pin	Connection
VCC	Supply voltage
VOUT	MCP3204 Channel 0
GND	Ground

    Always verify the physical pin orientation against the datasheet/package drawing before wiring the sensor.

Potentiometer — Voltage Measurement

The second ADC channel is used for voltage monitoring.

A potentiometer is connected to MCP3204 ADC Channel 1.

The potentiometer acts as a variable analog voltage source. By rotating the potentiometer, the voltage applied to ADC Channel 1 can be varied.

             +VCC
               │
               │
          ┌────┴────┐
          │   POT   │
          └────┬────┘
               │
               │ Variable Voltage
               ▼
        MCP3204 ADC CH1
               │
               │ SPI
               ▼
          LPC21xx MCU
               │
               ▼
        Voltage Display

    The potentiometer is used as a variable voltage input for demonstration. For real voltage measurement, an appropriate voltage-divider or voltage-sensing circuit should be used so that the ADC input never exceeds its permitted voltage range.

Temperature Monitoring and Threshold Alert

The LM35DZ/NOPB temperature sensor is connected to MCP3204 ADC Channel 0.

The MCP3204 converts the analog temperature signal into a digital value. The LPC21xx reads this value through SPI and calculates the temperature.

The measured temperature is then compared with a predefined temperature threshold.
Normal Temperature Condition

If the temperature is less than or equal to the threshold level, the system displays the measured temperature on the LCD.

Example:

Temperature = 25°C
Threshold   = 30°C

LCD:

Temperature
25.0 C

No GSM alert is generated because the temperature is within the permitted range.
High Temperature Condition

If the measured temperature becomes greater than the threshold level, the system identifies a high-temperature condition.

The LCD displays the high-temperature condition.

The LPC21xx then sends an alert through UART to the GSM modem.

The GSM modem sends the alert message to the configured mobile phone.

Example:

Temperature = 35°C
Threshold   = 30°C

LCD:

HIGH TEMP!
35.0 C

Communication Flow

LM35
 │
 ▼
MCP3204 ADC CH0
 │
 ▼
SPI
 │
 ▼
LPC21xx
 │
 ├── Temperature ≤ Threshold
 │       │
 │       ▼
 │      LCD
 │   Display Value
 │
 └── Temperature > Threshold
         │
         ├──► LCD
         │    HIGH TEMP
         │
         ▼
       UART0
         │
         ▼
     GSM Modem
         │
         ▼
   Alert SMS Message
         │
         ▼
    User Mobile

Temperature Alert Flow

                    Start
                      │
                      ▼
                Read LM35
                      │
                      ▼
              ADC Channel 0
                      │
                      ▼
                 SPI Read
                      │
                      ▼
               LPC21xx MCU
                      │
                      ▼
          Calculate Temperature
                      │
                      ▼
            Compare Threshold
                 /       \
                /         \
               ▼           ▼
        Temperature      Temperature
        ≤ Threshold      > Threshold
             │                │
             ▼                ▼
       Display Value      Display Alert
          on LCD             on LCD
                              │
                              ▼
                            UART
                              │
                              ▼
                         GSM Modem
                              │
                              ▼
                        User Mobile

Example GSM Alert

When the measured temperature exceeds the threshold, an SMS can be sent to the configured mobile number.

ALERT!

Temperature is above the threshold level.

Current Temperature: 35°C

The communication sequence is:

LPC21xx → UART → GSM Modem → Mobile Phone

The LPC21xx does not directly send the SMS. It communicates with the GSM modem through UART, and the GSM modem performs the mobile-network communication.
Complete Telemetry Flow

                         LPC21xx
                            │
                    ┌───────┴───────┐
                    │               │
                    ▼               ▼
                MCP3204 ADC       Keypad
                    │               │
             ┌──────┴──────┐        │
             │             │        ▼
             ▼             ▼     Password
           CH0            CH1    Authentication
             │             │
             ▼             ▼
           LM35       Potentiometer
             │             │
             ▼             ▼
       Temperature       Voltage
             │
             ▼
       Compare Threshold
             │
       ┌─────┴─────┐
       │           │
     Normal       High
       │           │
       ▼           ▼
      LCD         LCD
    Display     High Temp
                   │
                   ▼
                 UART
                   │
                   ▼
              GSM Modem
                   │
                   ▼
             Mobile Alert

User Authentication

The system uses a keypad-based authentication mechanism.

A six-digit password is entered through the keypad.

                Power ON
                   │
                   ▼
            System Initialization
                   │
                   ▼
             Password Entry
                   │
                   ▼
             Keypad Input
                   │
                   ▼
          Compare Stored Password
              /             \
             /               \
          Correct          Incorrect
             │                 │
             ▼                 ▼
      Access Granted          Retry
             │
             ▼
       Telemetry Menu

The password is stored/read using the I²C interface.

Password characters are masked on the LCD during entry.
Telemetry Selection

After successful authentication, the user can select the required telemetry parameter using the keypad.

0 → Temperature
1 → Voltage

Temperature

Keypad
   │
   ▼
Select Temperature
   │
   ▼
ADC CH0
   │
   ▼
LM35
   │
   ▼
Temperature
   │
   ▼
LCD / Threshold Alert

Voltage

Keypad
   │
   ▼
Select Voltage
   │
   ▼
ADC CH1
   │
   ▼
Potentiometer
   │
   ▼
Voltage
   │
   ▼
LCD Display

Hardware Components
Component	Part Number / Type	Connection	Purpose
Microcontroller	LPC21xx ARM7	—	Main controller
Temperature Sensor	LM35DZ/NOPB	MCP3204 CH0	Temperature measurement
ADC	MCP3204	SPI	Analog-to-digital conversion
Potentiometer	Variable resistor	MCP3204 CH1	Voltage input
LCD	16×2 LCD	Controller interface	Display
Keypad	4×4 Matrix Keypad	Controller interface	Password/menu input
I²C Memory	EEPROM	I²C	Password storage
GSM Modem	UART-compatible GSM module	UART0	SMS notification
LEDs	Standard LEDs	GPIO	Status indication
Motor	DC motor/control circuit	GPIO	Critical-condition response
Power Supply	Regulated DC supply	—	System power
Peripheral Communication
I²C

I²C is used for communication with the external memory used for password storage and associated I²C devices.

The application reads/writes the password through the I²C interface.
SPI

SPI0 is used to communicate between the LPC21xx and the MCP3204 ADC.

The ADC channels are:

SPI/ADC Channel 0 → LM35 → Temperature

SPI/ADC Channel 1 → Potentiometer → Voltage

UART

UART0 is used for communication between the LPC21xx and GSM modem.

LPC21xx
   │
 UART0
   │
   ▼
GSM Modem
   │
   ▼
Mobile Network
   │
   ▼
User Mobile

UART is responsible for transferring the alert information from the microcontroller to the GSM modem.
GPIO Control

The system uses GPIO outputs for LED indication and motor/control operation.

Example definitions used in the source:

#define led1   1<<18
#define led2   1<<19
#define led3   1<<20
#define motor1 1<<21
#define motor2 1<<22

These outputs can be used to indicate different temperature conditions and activate the motor/control circuit during a critical condition.
Software Architecture

main.c
│
├── System Initialization
│   ├── I²C
│   ├── LCD
│   ├── SPI
│   ├── UART
│   └── GPIO
│
├── Password Authentication
│   ├── Keypad
│   └── I²C Memory
│
├── Telemetry Selection
│
├── Temperature Monitoring
│   └── LM35 → ADC CH0
│
├── Voltage Monitoring
│   └── Potentiometer → ADC CH1
│
├── Threshold Checking
│
├── LCD Display
│
├── LED/Motor Control
│
└── GSM Alert
    └── UART → GSM → Mobile

Source Files

Restricted-Access-Embedded-Console-for-Multi-Channel-Telemetry/
│
├── main.c
├── keypad_i2c_define.h
├── spi.h
├── delay.h
└── decleration.h

main.c

Contains the main application logic:

    System initialization
    Password authentication
    Keypad handling
    LCD display
    Temperature monitoring
    Voltage monitoring
    Threshold checking
    LED control
    Motor/control logic
    UART/GSM communication

keypad_i2c_define.h

Contains functions related to:

    I²C communication
    Keypad interface
    LCD initialization
    LCD commands/data
    Display functions

spi.h

Contains SPI0 and ADC communication routines.

The MCP3204 ADC is used to acquire:

CH0 → LM35 temperature
CH1 → Potentiometer voltage

delay.h

Contains delay routines used by the embedded application.
decleration.h

Contains declarations and definitions used by the project.
Requirements
Hardware

    LPC21xx ARM7 development board
    LM35DZ/NOPB temperature sensor
    MCP3204 ADC
    Potentiometer
    16×2 LCD
    4×4 matrix keypad
    I²C EEPROM/memory
    GSM modem
    LEDs
    Motor/control circuitry
    Suitable regulated power supply
    Connecting wires and supporting components

Software

    Embedded C compiler
    LPC21xx-compatible IDE
    ARM7 development environment
    Programmer/debugger suitable for the target LPC21xx

Installation and Setup

Clone the repository:

git clone https://github.com/lokeshloka/Restricted-Access-Embedded-Console-for-Multi-Channel-Telemetry.git

cd Restricted-Access-Embedded-Console-for-Multi-Channel-Telemetry

Then:

    Open the source code in an LPC21xx-compatible development environment.
    Configure the target LPC21xx microcontroller.
    Connect the LM35DZ/NOPB output to MCP3204 Channel 0.
    Connect the potentiometer output to MCP3204 Channel 1.
    Connect the MCP3204 to the LPC21xx through SPI.
    Connect the LCD.
    Connect the keypad.
    Connect the I²C memory.
    Connect the GSM modem to UART0.
    Connect LEDs and motor/control outputs.
    Configure the password.
    Configure the GSM destination number.
    Build the firmware.
    Flash the firmware to the LPC21xx.
    Power on the system.
    Enter the password using the keypad.
    Select temperature or voltage monitoring.

Typical System Operation

Power ON
   │
   ▼
Initialize Peripherals
   │
   ▼
Password Authentication
   │
   ├── Incorrect → Retry
   │
   └── Correct
          │
          ▼
     Access Granted
          │
          ▼
    Select Telemetry
       /          \
      /            \
     ▼              ▼
Temperature        Voltage
     │                │
     ▼                ▼
   ADC CH0          ADC CH1
     │                │
     ▼                ▼
    LM35        Potentiometer
     │                │
     ▼                ▼
Temperature         Voltage
     │                │
     ▼                ▼
 Threshold          LCD
   Check           Display
     │
 ┌───┴────┐
 │        │
Normal   High
 │        │
 ▼        ▼
LCD      LCD
 │        │
 │        ▼
 │      UART
 │        │
 │        ▼
 │    GSM Modem
 │        │
 │        ▼
 │   Mobile Alert
 │
 ▼
Continue Monitoring

Temperature Threshold Example

Assume the configured threshold is 30°C.
Temperature	Condition	LCD	GSM Alert
20°C	Normal	Temperature displayed	No
25°C	Normal	Temperature displayed	No
30°C	Threshold	Temperature displayed	No
31°C	High	High-temperature indication	Yes
35°C	High	High-temperature indication	Yes
40°C	High/Critical	Alert indication	Yes

The threshold value can be changed in the software according to the application requirements.
Applications

This project can be used as a prototype for:

    Industrial temperature monitoring
    Restricted-access monitoring systems
    Remote equipment monitoring
    GSM-based alert systems
    Embedded telemetry systems
    Sensor monitoring applications
    Industrial safety systems
    Temperature-based automatic control
    ARM7 embedded-system projects

Security Considerations

For production deployment, the prototype should be improved with:

    Secure password storage
    Password hashing
    Login attempt limits
    Account lockout
    Removal of hard-coded phone numbers
    Configurable GSM settings
    Sensor fault detection
    ADC range validation
    Secure firmware protection
    Hardware protection for motor/control outputs

Do not commit real passwords, phone numbers, GSM credentials, or other sensitive information to the repository.
Future Enhancements

    🔐 Secure password hashing
    👤 Multiple user access levels
    ⏱️ Login lockout after failed attempts
    📡 IoT/cloud telemetry
    📱 Mobile application
    📊 Real-time temperature and voltage graphs
    🗄️ Telemetry data logging
    🚨 Configurable threshold values
    🔔 SMS and email notifications
    🧪 Sensor calibration
    🔧 Sensor fault detection
    🔒 Encrypted communication
    🌐 Web-based monitoring dashboard
    ⚙️ LCD/keypad-based system configuration

Project Objective

The objective of this project is to develop a restricted-access embedded telemetry console capable of monitoring multiple analog parameters and providing both local and remote notifications.

The project combines:

Authentication + Temperature Monitoring + Voltage Monitoring + LCD Display + Threshold Detection + UART + GSM Alert
Temperature Signal Path

LM35
  ↓
MCP3204 CH0
  ↓
SPI
  ↓
LPC21xx
  ↓
LCD / Threshold Check
  ↓
UART
  ↓
GSM
  ↓
Mobile Alert

Voltage Signal Path

Potentiometer
  ↓
MCP3204 CH1
  ↓
SPI
  ↓
LPC21xx
  ↓
LCD

This project demonstrates practical integration of:

    Embedded C
    ARM7/LPC21xx
    GPIO
    I²C
    SPI
    ADC
    UART
    GSM
    LCD
    Keypad
    LM35 temperature sensing
    Potentiometer-based voltage sensing
    Threshold-based monitoring
    Remote alert generation

Repository

Restricted Access Embedded Console for Multi-Channel Telemetry
Author

Lokesh Loka
License

No license is currently specified in the repository. Add an appropriate open-source license if you intend to distribute or modify the project.
Disclaimer

This project is intended for educational and experimental embedded-systems applications. Verify all electrical connections, ADC scaling, sensor specifications, GSM configuration, and actuator-control circuits against the actual hardware and manufacturer datasheets before deployment.
