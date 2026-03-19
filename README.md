# TCL Air Conditioner ESPHome ESP-IDF Controller

> Native ESPHome ESP-IDF firmware for TCL air conditioners with Home Assistant integration
> Simple DIY project with ESP32-C3/C6. Works directly with Home Assistant via native API.

---

## Features

- **ESPHome ESP-IDF**: Native ESP-IDF framework with ESPHome integration
- **Home Assistant**: Direct integration via Home Assistant API
- **UART Communication**: 9600 Baud, Even Parity protocol
- **OLED Display**: Optional SSD1306 128x32 display support
- **Status LEDs**: Optional TX/RX activity indication

---

## What You Need

### Recommended Hardware

| Board | Price | Link |
|-------|-------|------|
| **ESP32-C3-DevKitM-1** | ~8€ | [Espressif](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32c3/esp32-c3-devkitm-1/index.html) |
| **ESP32-C6-DevKitC-1** | ~10€ | [Espressif](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32c6/esp32-c6-devkitc-1/index.html) |

### Additional Requirements

- **USB-A plug or cable** for connection to the air conditioner
  Example: [AliExpress Link](https://www.aliexpress.com/item/1005005776162012.html)
- **SSD1306 OLED Display** (optional, 128x32, I2C)
- **Status LEDs** (optional, 5mm)

---

## Wiring

### USB-A to ESP32

| USB-A Pin | Wire Color | ESP32 Pin |
|-----------|------------|-----------|
| GND       | Black      | GND       |
| D+        | Green      | RX (GPIO3)|
| D-        | White      | TX (GPIO1)|
| VBUS      | Red        | VIN (5V)  |

### Default Pin Assignment

| Function | GPIO | Description |
|----------|------|-------------|
| UART TX  | GPIO1 | Data to AC  |
| UART RX  | GPIO3 | Data from AC|
| I2C SDA  | GPIO0 | OLED Display (optional) |
| I2C SCL  | GPIO2 | OLED Display (optional) |
| RX LED   | GPIO6 | Receive LED (optional) |
| TX LED   | GPIO4 | Transmit LED (optional) |

---

## Quick Start

### 1. Prerequisites

```bash
# Install ESPHome
pip install esphome

# Or use ESPHome Dashboard
```

### 2. Flash Firmware

```bash
# Clone repository
git clone https://github.com/sorz2122/tclac.git
cd tclac

# Connect device
# Make sure ESP32 is connected via USB

# Flash firmware
esphome run TCL-Conditioner.yaml
```

### 3. Home Assistant Integration

After flashing, the device will appear in Home Assistant automatically:

1. Home Assistant will discover the device via mDNS
2. Accept the connection using the API key
3. Control your AC via the Climate entity

---

## Configuration

### Edit TCL-Conditioner.yaml

```yaml
substitutions:
  device_name: tclac          # Unique device name
  humanly_name: TCL AC        # Display name in Home Assistant
  wifi_ssid: !secret wifi_ssid
  wifi_password: !secret wifi_password
  api_key: <your-api-key>     # Generate at esphome.io
  ota_pass: <ota-password>
  uart_rx: GPIO3
  uart_tx: GPIO1
```

### Optional Packages

Uncomment in `TCL-Conditioner.yaml`:

```yaml
packages:
  remote_package:
    url: https://github.com/sorz2122/tclac.git
    ref: master
    files:
      - packages/core.yaml
      - packages/leds.yaml      # Enable LEDs
      - packages/screen.yaml    # Enable OLED display
    refresh: 30s
```

---

## Supported Functions

| Function | Status | Home Assistant Entity |
|----------|--------|----------------------|
| Power On/Off | Yes | climate.power |
| Modes (Auto/Cool/Heat/Dry/Fan) | Yes | climate.mode |
| Temperature (16-31 C) | Yes | climate.temperature |
| Fan Speed (Auto/Quiet/Low/Med/High) | Yes | climate.fan_mode |
| Swing (Off/Vertical/Horizontal/Both) | Yes | climate.swing_mode |
| Display On/Off | Yes | switch.display |
| Beeper On/Off | Yes | switch.beeper |
| Eco/Sleep/Comfort Preset | Yes | climate.preset |

---

## OLED Display

The optional OLED display shows:

- Current time
- AC mode (OFF/AUTO/COOL/HEAT/DRY/FAN)
- WiFi signal strength
- Status indicators (Beeper, Display)

---

## Troubleshooting

### No Connection to AC

```
[TCL] Wrong byte
[TCL] Invalid checksum
```

**Solutions:**
- Check TX/RX wiring (TX to RX, RX to TX)
- Check cable continuity
- Verify 9600 Baud, Even Parity settings
- Enable debug logging

### WiFi Connection Issues

```
WiFi disconnected
```

**Solutions:**
- Check WiFi credentials
- Move device closer to router
- Check WiFi signal strength

---

## Compatible Air Conditioners

| Manufacturer | Models |
|--------------|--------|
| **TCL** | TAC-07CHSA, TAC-09CHSA, TAC-12CHSA, TAC-12CHDA, TAC-12CHFA |
| **Daichi** | AIR20AVQ1, AIR25AVQS1R-1, DA35EVQ1-1 |
| **Axioma** | ASX09H1, ASB09H1 |
| **Dantex** | RK-12SATI, RK-12SATIE |

**Note:** Not all models have an accessible UART port. Check before buying!

---

## Architecture

```
+----------------------------------+
|       Home Assistant             |
|       Native API                 |
+----------------+-----------------+
                 |
                 | WiFi
                 |
+----------------v-----------------+
|         ESPHome Device           |
|  +-------------+---------------+ |
|  |  TCL Climate Component      | |
|  |  +---------+  +-----------+ | |
|  |  | Protocol|  |   UART    | | |
|  |  | Handler |  |  Handler  | | |
|  |  +---------+  +-----------+ | |
|  +-------------+---------------+ |
+----------------+-----------------+
                 |
                 | UART (9600, Even)
                 |
+----------------v-----------------+
|      TCL Air Conditioner         |
|      (via USB/UART)              |
+----------------------------------+
```

---

## Credits

Original ESPHome Component:
- Miguel Angel Lopez (Original)
- xaxexa (Modifications)
- Nightingale with soldering iron (Refactoring & Component)

ESPHome ESP-IDF Edition:
- sorz2122

---

## Support

If you like this project, support the development:

https://buymeacoffee.com/sorz2122

---

## License

Licensed under MIT License.

---

## Contributing

Contributions are welcome!

1. Create a fork
2. Feature branch: `git checkout -b feature/NewFeature`
3. Commit: `git commit -am 'Add feature'`
4. Push: `git push origin feature/NewFeature`
5. Create Pull Request

---

**Made with for the Smart Home Community**
