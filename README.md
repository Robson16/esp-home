# ESP-Home Sensor Network Project

## Overview

This project manages a distributed IoT sensor network using ESP32 microcontrollers with ESPHome. The system consists of independent remote sensor nodes that communicate directly with a central Home Assistant instance for environmental monitoring.

### Architecture 

```text
┌────────────────────────────────────────────────┐
│                 Home Assistant                 │
│          (Raspberry Pi Docker Swarm)           │
└──────┬──────────────────────────────────┬──────┘
       │                                  │
┌──────▼──────┐                    ┌──────▼──────┐
│   Sensor    │                    │   Sensor    │
│   Node 01   │                    │   Node 02   │
│  Temp,      │                    │  Temp,      │
│  Humidity   │                    │  Humidity   │
└─────────────┘                    └─────────────┘
```

### Components

*   **Sensor Nodes** (`esphome-sensor-node-01.yaml` / `02.yaml`): Remote data collectors for environmental monitoring in different rooms.

#### Current Features

*   WiFi connectivity with fallback AP
*   Native Home Assistant API integration
*   Web interface for local access
*   Status LED with diagnostic blink patterns (GPIO 2)
*   Local OLED display for real-time data readout
*   Secure OTA (Over-The-Air) updates

#### Planned Features

*   Temperature & Humidity sensors (DHT11/22, BME280)
*   Air quality sensor (CO2, PM2.5)
*   Motion/Presence detection

### Technical Specs

*   **Board:** ESP32-DEV
*   **Framework:** ESP-IDF
*   **Minimum ESPHome Version:** 2025.11.0

## Hardware Reference

*   ESP32 Pinout

## File Structure

```text
esp-home/
├── esphome-sensor-node-01.yaml  # Node 1 configuration
├── esphome-sensor-node-02.yaml  # Node 2 configuration
├── secrets.yaml                 # WiFi and API credentials (not versioned)
└── README.md                    # This file
```

## Setup Instructions

### Prerequisites

*   ESPHome installed (CLI or Docker Dashboard)
*   Home Assistant instance
*   ESP32 Development Boards
*   USB Cable (Data transfer capable)
*   Chromium-based browser (Chrome/Edge/Brave) for Web Flashing

### Installation Steps

1.  **Clone or Download This Project**
```bash
git clone <repository-url>
cd esp-home
```

2.  **Configure Secrets**
    Create or edit `secrets.yaml` in the root folder:

```yaml
wifi_ssid: "YOUR_WIFI_SSID"
wifi_password: "YOUR_WIFI_PASSWORD"

# Node 01 Secrets
node1_api_encryption_key: "GENERATED_KEY"
node1_ota_password: "SECURE_PASSWORD"
node1_fallback_ap_password: "SECURE_PASSWORD"

# Node 02 Secrets
node2_api_encryption_key: "GENERATED_KEY"
node2_ota_password: "SECURE_PASSWORD"
node2_fallback_ap_password: "SECURE_PASSWORD"
```
**IMPORTANT**: Never commit `secrets.yaml` to version control!

3.  **Flash the Nodes (Via Web Dashboard)**
    The easiest way to flash locally is using the ESPHome Web Dashboard:

    Start the local server:

```bash
esphome dashboard .
```

Open `http://localhost:6052` in Chrome or Edge.

*   Connect the ESP32 via USB.
*   Click **Install** on the desired node card, select **Plug into this computer**, and choose the COM port.
*   (Note: You may need to hold the physical "BOOT" button on the ESP32 when the terminal shows `Connecting...`)

### Alternative: Flash via CLI

```bash
esphome run esphome-sensor-node-01.yaml
```

## Sensor Integration

Add sensor configurations to your node's YAML file.

#### Example: Temperature & Humidity (DHT11)

```yaml
sensor:
  - platform: dht
    pin: 4
    model: DHT11
    temperature:
      name: "Room Temperature"
      id: temp_sensor
    humidity:
      name: "Room Humidity"
      id: humidity_sensor
    update_interval: 60s
```

#### Example: OLED Display (SSD1306)

Para exibir dados em um display local, adicione os componentes `font` e `display`. Este exemplo usa um display I2C SSD1306 e assume que o barramento I2C já está configurado.

```yaml
font:
  # Configuração de fonte dupla para displays bicolores (amarelo/azul)
  - file: "gfonts://Roboto"
    id: font_header
    size: 12
  - file: "gfonts://Roboto"
    id: font_data
    size: 18

display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    address: 0x3C
    lambda: |-
      it.print(0, 0, id(font_header), "MONITOR AMBIENTAL");
      if (id(temp_sensor).has_state()) {
        it.printf(0, 20, id(font_data), "Temp: %.1f C", id(temp_sensor).state);
      } else {
        it.print(0, 20, id(font_data), "Temp: Lendo...");
      }
      if (id(humidity_sensor).has_state()) {
        it.printf(0, 42, id(font_data), "Umid: %.1f %%", id(humidity_sensor).state);
      } else {
        it.print(0, 42, id(font_data), "Umid: Lendo...");
      }
```

## Home Assistant Integration

1.  Go to **Settings → Devices & Services** in Home Assistant.
2.  The nodes should be auto-discovered via mDNS. Click **Configure**.
3.  If not discovered, click **+ Add Integration**, search for "ESPHome", and enter the node's IP address.
4.  Provide the `api_encryption_key` from your `secrets.yaml` when prompted.

## Testing & Diagnostics

### LED Blink Patterns (Test Mode)

The built-in LED (GPIO 2) can be used for diagnostics:

*   **Slow Blink** (1s on / 1s off): Normal operation / Boot successful
*   **Fast Blink** (100ms on / 100ms off): Data transmission test
*   **Alert Pattern** (200ms on/off): System alert state

## Troubleshooting

### Compilation Fails on Windows (C++ Linker / Cache Errors)

If you add or remove major components (like sensors) and get `undefined reference` errors during compilation, clear the build cache:

```bash
esphome clean esphome-sensor-node-01.yaml
```
Then, run the install command again.

### Device Not Connecting to WiFi

*   Check SSID and password in `secrets.yaml`.
*   Check router WiFi channel (try 1, 6, or 11 for 2.4GHz).
*   Review serial logs: `esphome logs esphome-sensor-node-01.yaml`.

### Home Assistant Not Discovering Device

*   Ensure mDNS is working on your network.
*   Verify the API encryption key matches exactly.

## Advanced Features (Future)

### Wireless Updates Over-The-Air (OTA)

After the initial USB flash, all future updates can be done wirelessly via the Dashboard or CLI:

```bash
esphome run esphome-sensor-node-01.yaml --device <NODE_IP_ADDRESS>
```

### Automations

```yaml
automation:
  - alias: "Alert on High Temperature"
    trigger:
      platform: numeric_state
      entity_id: sensor.room_temperature
      above: 30
    action:
      service: light.turn_on
      entity_id: light.sensor_node_status_led
      data:
        effect: "Alert Pattern"
```

## License

This project is provided as-is for personal IoT home automation use.

**Last Updated:** March 2026
**ESPHome Version:** 2025.11.0+
**Status:** Active Development
