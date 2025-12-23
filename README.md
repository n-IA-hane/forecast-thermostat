# Forecast-Thermostat v4.0

A smart ESP32-based thermostat with ILI9341 touchscreen display, featuring 5-day weather forecasts, Home Assistant integration, and power consumption monitoring.

![aztouch_wall](readme_img/aztouch%20wall.jpg)

**Compatible Hardware:** [AZTouch on Amazon](https://www.amazon.it/AZDelivery-AZ-Touch-custodie-pollici-ESP8266/dp/B081FC31Q5?th=1)

---

## Features

- **5-Day Weather Forecast** - Displays weather conditions, temperatures, and precipitation directly fetched from Home Assistant
- **Smart Thermostat Control** - Bang-bang temperature control with touchscreen interface
- **Power Consumption Monitoring** - Real-time display of household energy usage with high consumption alerts
- **Alarm Control Panel** - Integrated page for arming/disarming your Home Assistant alarm
- **Motion-Activated Backlight** - Auto-dimming display with PIR sensor wake-up
- **RTTTL Buzzer Support** - Play notification melodies via Home Assistant service calls
- **Offline Operation** - Core thermostat functionality works even without internet connection
- **Fully Localized** - Italian day/month names and weather descriptions (easily customizable)

---

## What's New in v4.0

### Direct Weather Forecast Integration

**No more template sensors required!** Previous versions required creating a template sensor in Home Assistant to fetch weather forecasts. Version 4.0 calls the `weather.get_forecasts` service directly from the ESP32.

The device now:
- Automatically fetches forecasts when connecting to Home Assistant (with 5s delay for stability)
- Updates forecasts every 5 minutes (configurable via `forecast_interval` substitution)
- Handles all JSON parsing on-device

### Other Improvements

- Professional English documentation throughout the codebase
- Cleaner code structure with organized sections
- Improved error handling for forecast fetching
- Removed dependency on external Home Assistant sensors for forecasts

---

## Hardware Requirements

| Component | Description | Image |
|-----------|-------------|-------|
| **ESP32 + Display** | AZTouch kit with ILI9341 320x240 TFT and XPT2046 touch controller | ![aztouch](readme_img/aztouch.jpg) |
| **BME280** | Temperature, humidity, and pressure sensor (I2C, address 0x76) | ![bme280](readme_img/bme280.jpg) |
| **PIR Sensor** | Motion sensor for backlight activation (optional) | ![pir](readme_img/rclw-0516.jpg) |

### Pin Configuration

| Function | GPIO |
|----------|------|
| Display CS | 5 |
| Display DC | 4 |
| Display Reset | 22 |
| Touch CS | 14 |
| SPI CLK | 18 |
| SPI MOSI | 23 |
| SPI MISO | 19 |
| I2C SDA | 32 |
| I2C SCL | 25 |
| Backlight PWM | 15 |
| PIR Sensor | 26 |
| Buzzer | 21 |

---

## Home Assistant Entities

### Required

| Entity | Description |
|--------|-------------|
| `weather.casa` | Any Home Assistant weather integration (Met.no, OpenWeatherMap, etc.) |
| `switch.shelly_caldaia` | Switch entity controlling your heater/boiler |

### Optional

| Entity | Description |
|--------|-------------|
| `sensor.consumo_totale_power` | Total household power consumption (W) |
| `sensor.consumo_cantina_power` | Secondary power sensor, e.g., washing machine (W) |
| `alarm_control_panel.allarme` | Home Assistant alarm control panel |

---

## Installation

### 1. Prepare Configuration Files

Clone this repository and update `secrets.yaml` with your WiFi credentials:

```yaml
wifi_ssid: "YourWiFiSSID"
wifi_password: "YourWiFiPassword"
fallpass: "FallbackAPPassword"
```

### 2. Customize Entity IDs

Edit the `substitutions` section in `forecast-thermostat-32x24.yaml` to match your Home Assistant entities:

```yaml
substitutions:
  heater: switch.shelly_caldaia             # Your heater switch
  weather_entity: weather.casa              # Your weather entity
  forecast_interval: 5min                   # Forecast update interval
  tc: sensor.consumo_totale_power           # Total power sensor (optional)
  pc: sensor.consumo_cantina_power          # Secondary power sensor (optional)
  alarm_entity: alarm_control_panel.allarme # Alarm panel (optional)
  icon_xy: '60x60'                          # Weather icon size
```

### 3. Compile and Flash

Using ESPHome CLI:
```bash
esphome run forecast-thermostat-32x24.yaml
```

Or upload via ESPHome Dashboard in Home Assistant.

### 4. Integration

The device will automatically appear in Home Assistant's ESPHome integration. No additional configuration required - forecasts are fetched directly via the API.

---

## Usage

### Display Pages

The thermostat has multiple display pages accessible via touch navigation:

**Main Thermostat Page:**
- Current date and time
- Outdoor weather with icon and temperature
- Indoor temperature from BME280
- Humidity (indoor/outdoor) and wind speed
- 5-day weather forecast with icons
- Thermostat status and target temperature
- Power consumption indicators

**Alarm Control Page:**
- Arm Home / Arm Away / Disarm buttons
- Current alarm state with status icon

### Touchscreen Controls

| Action | Location | Function |
|--------|----------|----------|
| **Short tap** | Center bottom | Enable HEAT mode |
| **Long press (>0.5s)** | Center bottom | Disable thermostat (OFF) |
| **Tap** | Right bottom (+) | Increase target temperature by 0.5°C |
| **Tap** | Left bottom (-) | Decrease target temperature by 0.5°C |
| **Tap** | Top right | Next page |
| **Tap** | Top left | Previous page |

### Backlight Behavior

- **Motion detected:** Full brightness for 20 seconds
- **After 20s:** Dims to 50% brightness
- **After 60s total:** Display turns off
- **Any touch:** Resets the timer to full brightness

---

## Home Assistant Integration

Once flashed, the thermostat appears automatically in Home Assistant as an ESPHome device. You can control the target temperature, switch heating modes, and monitor sensor readings directly from the HA dashboard.

![hass_integration](readme_img/hass_thermostat.jpg)

---

## RTTTL Buzzer

The AZTouch board includes a buzzer for audio notifications. Play melodies via Home Assistant service call:

```yaml
service: esphome.forecast_thermostat_32x24_play_rtttl
data:
  song_str: "siren:d=8,o=5,b=100:d,e,d,e,d,e,d,e"
```

---

## Demo Video

https://github.com/n-IA-hane/forecast-thermostat/raw/master/readme_img/video.mp4

---

## Removing Optional Features

### Without Alarm Panel

Set a dummy value for the substitution and remove the related components:

```yaml
substitutions:
  alarm_entity: alarm_control_panel.unused  # Dummy value
```

Then remove or comment out:
- `touch_key_home`, `touch_key_away`, `touch_key_disarm` binary sensors
- `stato_allarme` text sensor
- `alarm_cp_page` display page

### Without Power Monitoring

Set dummy values for the substitutions and remove the related components:

```yaml
substitutions:
  tc: sensor.unused_power    # Dummy value
  pc: sensor.unused_power    # Dummy value
```

Then remove or comment out:
- `consumo_t` and `consumo_c` sensors
- Sections 10 and 11 in the thermostat page lambda

---

## Troubleshooting

### Forecasts not displaying

1. Check that your weather entity is working in Home Assistant
2. Verify the `weather_entity` substitution matches your entity ID
3. Check ESPHome logs for forecast-related errors
4. Ensure Home Assistant API connection is established (5s delay after connection)

### Touch calibration issues

Adjust the calibration values in the touchscreen section:
```yaml
calibration:
  x_min: 380
  x_max: 3800
  y_min: 370
  y_max: 3900
```

### BME280 not detected

- Verify I2C wiring (SDA to GPIO32, SCL to GPIO25)
- Check sensor I2C address (default 0x76, some modules use 0x77)

---

## Version History

| Version | Changes |
|---------|---------|
| **4.0** | Direct `weather.get_forecasts` integration, no template sensor required, auto-fetch on HA connection, professional documentation |
| **3.x** | Template sensor required for forecasts, 5-day display, alarm panel |
| **2.x** | Initial ESPHome port, basic thermostat functionality |

---

## License

This project is licensed under the GPL-3.0 License - see the [LICENSE](LICENSE) file for details.

---

## Contact

For feedback, feature requests, or issues:
- **GitHub Issues:** [Open an issue](https://github.com/n-IA-hane/forecast-thermostat/issues)
- **Email:** meconiotech@gmail.com

Enjoy your smart thermostat!
