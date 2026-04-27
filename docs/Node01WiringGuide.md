# 🌐 Wiring Guide - Sensor Node 01

## 1️⃣ I2C Bus (Shared)

These modules are connected in parallel. All share the same SDA and SCL pins.

**ESP32 Pins:** SDA = 21 | SCL = 22

### OLED SSD1306 Module (Display)

| Pin | Connection |
|-----|------------|
| VCC | 3.3V |
| GND | GND |
| SCL | GPIO 22 |
| SDA | GPIO 21 |

### BMP280 Module (Temp/Pressure)

| Pin | Connection |
|-----|------------|
| VCC | 3.3V |
| GND | GND |
| SCL | GPIO 22 |
| SDA | GPIO 21 |

---

## 2️⃣ SPI Bus (7-Segment Display)

The MAX7219 module uses unidirectional SPI communication.

### MAX7219 Module

| Pin | Connection | Note |
|-----|------------|------|
| VCC | 5V (VIN) | Consumes a lot of power when all LEDs are lit |
| GND | GND | |
| DIN (MOSI) | GPIO 23 | |
| CS (LOAD) | GPIO 5 | |
| CLK | GPIO 18 | |

---

## 3️⃣ Digital Sensors (3.3V Signal)

Sensors that send ready data packets or binary states (High/Low).

### DHT22 Module (Temp/Humidity)

| Pin | Connection |
|-----|------------|
| VCC | 3.3V |
| GND | GND |
| DATA / OUT | GPIO 4 |

### PIR Module (Presence/Motion)

| Pin | Connection | Note |
|-----|------------|------|
| VCC | 5V (VIN) | Most PIRs require 5V to operate, but the output signal is 3.3V (Safe) |
| GND | GND | |
| OUT / DATA | GPIO 27 | |

---

## 4️⃣ Analog Sensors (ADC - Voltage Reading)

These pins read voltage variation.

⚠️ **Warning:** ESP32 pins support a maximum of 3.3V on analog signal.

### LDR Module (Luminosity)

| Pin | Connection | Note |
|-----|------------|------|
| VCC | 3.3V | Power with 3.3V to ensure the maximum output never exceeds the ESP32 limit |
| GND | GND | |
| A0 (Analog Out) | GPIO 34 | |

### MQ-135 Module (Air Quality)

| Pin | Connection | Note |
|-----|------------|------|
| VCC | 5V (VIN) | The heating coil requires 5V |
| GND | GND | |
| A0 (Analog Out) | GPIO 33 | |

### MQ-2 Module (Flammable Gas)

| Pin | Connection |
|-----|------------|
| VCC | 5V (VIN) |
| GND | GND |
| A0 (Analog Out) | GPIO 35 |

### MQ-7 Module (Carbon Monoxide)

| Pin | Connection |
|-----|------------|
| VCC | 5V (VIN) |
| GND | GND |
| A0 (Analog Out) | GPIO 32 |

#### 📌 Hardware Note

Since MQ modules are powered with 5V, the A0 pin can theoretically output up to 5V at maximum gas saturation. For maximum safety of the ESP32 ADC port in the long term, engineers usually:

- Place a small **resistive voltage divider** between the MQ's A0 pin and the ESP32 GPIO, or
- Confirm in the specific datasheet of the board manufacturer if it has output limitation

---

## 5️⃣ Internal Pins (Do not require external wiring)

### Status LED (Blue built-in on the board)

- **GPIO:** 2
- **Usage:** Blinks when PIR detects motion (integrated in the script)