# ESP-Home Sensor Network Project

## Overview

This project manages a distributed IoT sensor network using ESP32 microcontrollers with ESPHome. The system consists of a central hub controller and multiple remote sensor nodes for environmental monitoring.

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│         Hub Controller (Central)                         │
│  - Zigbee Coordinator (planned)                          │
│  - Display/Touch Screen (planned)                        │
│  - Raspberry Pi Cluster Monitoring                       │
│  - Network Coordination                                  │
└──────────────────────┬──────────────────────────────────┘
                       │
         ┌─────────────┼─────────────┐
         │             │             │
    ┌────▼────┐  ┌────▼────┐  ┌────▼────┐
    │ Sensor  │  │ Sensor  │  │ Sensor  │
    │ Node 01 │  │ Node 02 │  │ Node NN │
    │ Temp,   │  │ Humidity│  │ Pressure│
    │ Humidity│  │ Motion  │  │ Air QA  │
    └─────────┘  └─────────┘  └─────────┘
```

## Components

### 1. **Hub Controller** (`esphome-hub.yaml`)

The central communication hub that coordinates the entire sensor network.

**Current Features:**
- WiFi connectivity with fallback AP
- Native Home Assistant API integration
- Web interface for local access
- Status LED with diagnostic blink patterns
- Secure OTA (Over-The-Air) updates

**Planned Features:**
- Zigbee coordinator for extended network reach
- Display/Touch screen interface
- Raspberry Pi cluster monitoring
- Real-time sensor data aggregation
- Advanced automation rules

**Technical Specs:**
- Board: ESP32 (generic)
- Framework: ESP-IDF
- Encryption: Enabled
- Min Version: 2025.11.0

---

### 2. **Sensor Node** (`esphome-sensor-node.yaml`)

Remote data collector for environmental monitoring.

**Current Features:**
- WiFi connectivity with fallback AP
- Native Home Assistant API integration
- Web interface for local access
- Status LED with diagnostic blink patterns
- Secure OTA (Over-The-Air) updates

**Planned Features:**
- Temperature sensor (DHT22, BME280, etc.)
- Humidity sensor
- Pressure sensor
- Air quality sensor (CO2, PM2.5, etc.)
- Motion/Presence detection
- Light intensity measurement
- Binary sensors (door/window contact, leak detection)

**Technical Specs:**
- Board: ESP32-DEV
- Framework: ESP-IDF
- Encryption: Enabled
- Min Version: 2025.11.0

---

## Hardware Reference

### ESP32 Pinout

![ESP32 Pinout](images/ESP32-Pinout.png)

---

## File Structure

```
esp-home/
├── esphome-hub.yaml           # Central hub configuration
├── esphome-sensor-node.yaml   # Sensor node template
├── secrets.yaml               # WiFi and API credentials (not versioned)
├── archive/                   # Old configurations
│   └── super-no-bancada.yaml  # Previous hub version
└── README.md                  # This file
```

---

## Setup Instructions 

### Prerequisites

- **ESPHome** installed (latest version)
- **Home Assistant** instance (optional but recommended)
- **ESP32 Development Board(s)**
- **USB Cable** for flashing
- **WiFi Network** credentials

### Installation Steps

#### 1. Clone or Download This Project

```bash
git clone <repository-url>
cd esp-home
```

#### 2. Configure Secrets

Create or edit `secrets.yaml`:

```yaml
wifi_ssid: "YOUR_WIFI_SSID"
wifi_password: "YOUR_WIFI_PASSWORD"

# Sensor Node Secrets
node_api_encryption_key: "GENERATED_KEY_FOR_NODE"
node_ota_password: "SECURE_PASSWORD_FOR_NODE_OTA"
node_fallback_ap_password: "SECURE_PASSWORD_FOR_NODE_AP"

# Hub Controller Secrets
hub_api_encryption_key: "GENERATED_KEY_FOR_HUB"
hub_ota_password: "SECURE_PASSWORD_FOR_HUB_OTA"
hub_fallback_ap_password: "SECURE_PASSWORD_FOR_HUB_AP"

# Password for 'admin' user on Hub web interface
web_password: "YOUR_WEB_INTERFACE_PASSWORD" 
```

**IMPORTANT:** Never commit `secrets.yaml` to version control!

#### 3. Flash the Hub Controller

```bash
esphome run esphome-hub.yaml
```

- Select your USB port when prompted
- Allow the firmware to compile and upload
- Monitor the logs for successful connection

#### 4. Flash Sensor Nodes

```bash
esphome run esphome-sensor-node.yaml
```

For multiple nodes, create copies with unique names:
- `esphome-sensor-node-01.yaml`
- `esphome-sensor-node-02.yaml`
- etc.

---

## Testing & Diagnostics

### LED Blink Patterns (Test Mode)

Both devices have status LEDs on **GPIO 2** with test patterns:

- **Slow Blink** (1s on / 1s off): Normal operation
- **Fast Blink** (100ms on / 100ms off): Data transmission test
- **Alert Pattern** (150-200ms on/off): System alert state

To enable a blink pattern:

1. Open Home Assistant or local web interface
2. Find "Hub Status LED" or "Sensor Node Status LED"
3. Select desired effect

### Web Interface

- **Hub Controller:** `http://192.168.X.X` (check your router for IP)
- **Sensor Node:** `http://192.168.X.X`

### Serial Monitor

For USB debugging:

```bash
esphome logs esphome-hub.yaml
esphome logs esphome-sensor-node.yaml
```

---

## Sensor Integration

Add sensor configurations:

### Example: Temperature & Humidity (DHT22)

```yaml
# Add to i2c or uart section
dht:
  - platform: dht
    pin: GPIO4
    model: DHT22
    id: dht_sensor

sensor:
  - platform: dht
    temperature:
      name: "Room Temperature"
      id: temp_sensor
    humidity:
      name: "Room Humidity"
      id: humidity_sensor
    update_interval: 60s
```

### Example: BME280 (Temperature, Humidity, Pressure)

```yaml
i2c:
  sda: GPIO21
  scl: GPIO22

sensor:
  - platform: bme280
    temperature:
      name: "Temperature"
    pressure:
      name: "Pressure"
    humidity:
      name: "Humidity"
```

---

## Home Assistant Integration

### Enable ESPHome Integration

1. Go to **Settings → Devices & Services**
2. Click **+ Create Automation**
3. Search for "ESPHome"
4. Follow the setup wizard
5. Devices should auto-discover via mDNS

### Manual Addition

Edit `configuration.yaml`:

```yaml
esphome:

mqtt:
  broker: localhost
```

---

## Security Considerations

### ✅ Best Practices

- **Encryption Keys**: Generate unique keys for each device
  ```bash
  esphome generate-encryption-key
  ```
- **Fallback AP Password**: Use strong, unique passwords
- **OTA Password**: Enable secure OTA with strong passwords
- **Web Interface**: Protect with strong credentials
- **WiFi**: Use WPA2 or WPA3 encryption
- **API Key**: Rotate periodically

### Secrets Management

```yaml
# ✅ GOOD: Use secrets
api:
  encryption:
    key: !secret api_encryption_key

# ❌ BAD: Hardcoded values
# Never commit sensitive data!
```

---

## Troubleshooting

### Device Not Connecting to WiFi

1. Check SSID and password in `secrets.yaml`
2. Verify fallback AP is enabled
3. Check router WiFi channel (try 1, 6, or 11 for 2.4GHz)
4. Review serial logs: `esphome logs esphome-hub.yaml`

### Unable to Flash Device

- Ensure USB cable is properly connected
- Try a different USB port or cable
- Hold BOOT button while connecting
- Verify correct board selection in YAML

### Home Assistant Not Discovering Device

- Ensure mDNS is working on your network
- Check Home Assistant logs for errors
- Manual IP assignment may be required
- Verify encryption key matches

### LED Not Blinking

- Check GPIO 2 availability (may conflict with boot mode)
- Verify hardware connections
- Check for output conflicts in YAML
- Review compilation errors

### OTA Authentication Failed

If you changed the OTA password in `secrets.yaml`, wireless updates will fail because the device still uses the old password.
**Solution:** Flash the device via USB cable to update the stored credentials.

---

## Advanced Features (Future)

### Wireless Updates Over-The-Air (OTA)

After initial USB flash, all updates via WiFi:

```bash
esphome run esphome-hub.yaml --device 192.168.1.100
```

### Climate Control Integration

```yaml
climate:
  - platform: thermostat
    name: "Smart Thermostat"
    sensor: dht_temp
    default_target_temperature: 22
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
      entity_id: light.hub_status_led
      data:
        effect: "Alert Pattern"
```

---

## Development Workflow

### Adding New Sensor

1. Create new device YAML file or update existing
2. Add sensor configuration block
3. Compile: `esphome compile esphome-sensor-node.yaml`
4. Flash: `esphome run esphome-sensor-node.yaml`
5. Monitor: `esphome logs esphome-sensor-node.yaml`

### Configuration Validation

```bash
esphome config esphome-hub.yaml
esphome config esphome-sensor-node.yaml
```

### Dirty Flash (Restore Factory Settings)

```bash
esphome run esphome-hub.yaml --device=/dev/ttyUSB0 --no-logs
```

---

## Documentation References

- [ESPHome Documentation](https://esphome.io/)
- [Home Assistant Integration](https://www.home-assistant.io/integrations/esphome/)
- [ESP32 Pinout Reference](https://randomnerdtutorials.com/esp32-pinout-reference-which-gpio-pins-are-safe-to-use/)
- [DHT Sensor Guide](https://esphome.io/components/sensor/dht.html)
- [I2C Protocol](https://esphome.io/components/i2c.html)

---

## License

This project is provided as-is for personal IoT home automation use.

---

## Support & Contribution

For issues, improvements, or sensor recommendations:

1. Check the troubleshooting section
2. Review ESPHome documentation
3. Check Home Assistant community forums
4. Monitor device logs for detailed error messages

---

**Last Updated:** January 29, 2026  
**ESPHome Version:** 2025.11.0+  
**Status:** Active Development
