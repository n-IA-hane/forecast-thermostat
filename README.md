# Forecast-Thermostat

This is the updated program for the AZTouch electronic board, developed using ESPHome to function as a thermostat with integrated weather forecast capabilities. Below, you will find the latest details, features, and instructions for configuring the device.

![aztouch_wall](https://github.com/niahane/meteo-thermostat/blob/7a2923ba1b6a4c6382c3172988ddbd63d4416278/readme_img/aztouch%20wall.jpg)
[AZTouch on Amazon](https://www.amazon.it/AZDelivery-AZ-Touch-custodie-pollici-ESP8266/dp/B081FC31Q5?th=1)

## Features

- **Weather Forecast Integration:** Fetches forecasts using Home Assistant's weather services.
- **Local Data Processing:** Parses forecast data directly on the ESP32 to avoid overloading Home Assistant.
- **Dynamic Display Logic:** The screen layout is now divided into a grid-like matrix using auto-calculated variables set during device initialization. This ensures flexible and efficient rendering.
- **Optimized Rendering:** The deserialization of JSON forecast data occurs only when the forecast state changes, minimizing CPU load during rendering. Most calculations are performed at startup or when entities change state, further reducing runtime computational overhead.
- **Expanded Forecast View:** Now displays 5 days of weather forecasts instead of 4, offering a more comprehensive view.
- **Minimal Database Usage:** Reduces sensor-related database writes in Home Assistant.
- **Touchscreen Support:** Fully interactive interface for thermostat control.
- **Independent Operation:** Thermostat logic is embedded in the ESP32, ensuring basic functionality even without internet access.
- **Additional Sensors:**
  - BME280 for internal temperature, humidity, and pressure.
  ![bme280](https://github.com/niahane/meteo-thermostat/blob/7a2923ba1b6a4c6382c3172988ddbd63d4416278/readme_img/bme280.jpg)
  - Motion sensor to control the backlight.
  ![rclw](https://github.com/niahane/meteo-thermostat/blob/7a2923ba1b6a4c6382c3172988ddbd63d4416278/readme_img/rclw-0516.jpg)
- **Optional Energy Monitoring:** Supports power consumption sensors for total and partial loads.
- **Customizable Localization:** Weather conditions, days, and months can be translated directly in the code.
- **Google Fonts Integration:** Fonts are now downloaded dynamically, eliminating the need for a local fonts folder.
- **Alarm Control Panel:** Includes a dedicated page for managing the alarm system. If unused, references to this page and related touchscreen controls can be removed.

## What's New?

1. **Updated Forecast Method:**
   - Home Assistant no longer provides forecast attributes directly within the `weather` entity. Instead, you must create a template sensor to call the `weather.get_forecasts` service.
   - Example configuration for Home Assistant:
     ```yaml
     - trigger:
         - platform: time_pattern
           minutes: /1
       action:
         - service: weather.get_forecasts
           data:
             type: daily
           target:
             entity_id: weather.casa
           response_variable: previsioni
       sensor:
         - name: Previsioni
           unique_id: weather_forecast
           state: "{{ now().isoformat() }}"
           attributes:
             forecast: "{{ previsioni['weather.casa']['forecast'] }}"
     ```
2. **Dynamic Display Logic:**
   - The screen layout uses a matrix grid system where positions are auto-calculated during initialization. This improves flexibility and simplifies UI adjustments.
3. **Optimized Rendering:**
   - JSON deserialization and other intensive computations are triggered only when forecast data or related entities change state.
   - Runtime CPU load is minimized by performing most calculations at startup or during state changes.
4. **Expanded Forecast View:**
   - The weather forecast now displays 5 days instead of 4, providing a more detailed view.
5. **Enhanced Logging:** Debug-level logs for easier troubleshooting and development.
6. **Optimized UI Design:** Clearer display with refined layouts for weather, thermostat, and energy monitoring data.
7. **Simplified Font Handling:** Fonts are now downloaded directly from Google Fonts, reducing setup complexity.
8. **Alarm Control Panel:** Added a dedicated page for managing the alarm system. If unnecessary, related code can be removed.

## Sensors and Entities

- **Required:**
  - `switch.shelly_caldaia`: Entity to control the heater (or equivalent switch).
  - `weather.casa`: Home Assistant weather entity.
- **Optional:**
  - `sensor.consumo_totale_power`: Sensor for total power consumption.
  - `sensor.consumo_cantina_power`: Sensor for partial power consumption.
  - `alarm_control_panel.allarme`: Alarm panel entity for integrating alarm controls.

## Installation

1. **Prepare ESPHome Configuration:**
   - Update the `secrets.yaml` file with your Wi-Fi credentials.
   - Modify the `substitutions` section in the YAML file to match your entities:
     ```yaml
     substitutions:
       heater: switch.shelly_caldaia
       weather_entity: weather.casa
       tc: sensor.consumo_totale_power
       pc: sensor.consumo_cantina_power
       alarm_entity: alarm_control_panel.allarme
     ```
2. **Compile and Flash:**
   - Compile the firmware and upload it to the ESP32 using ESPHome.
3. **Integrate with Home Assistant:**
   - Add the thermostat to Home Assistant as an ESPHome integration.
   - Ensure the weather forecast template sensor is set up correctly.

## Usage

### Touchscreen Controls
- **Turn On/Off Thermostat:** Tap the central power button.
- **Adjust Temperature:** Use the `+` and `-` buttons on the screen.
- **Disable Thermostat:** Press and hold the flame icon.
- **View Weather Forecast:** Displays a 5-day forecast with icons, descriptions, and temperatures.
- **Energy Monitoring:** If configured, view real-time power consumption data.
- **Alarm Integration:** Use the alarm control panel to arm/disarm the alarm system directly from the touchscreen.

### Notes
- The thermostat enforces a minimum on/off duration to protect the boiler from excessive wear.
- The motion sensor temporarily increases screen brightness when activity is detected.
- Weather forecasts are updated every minute using the configured Home Assistant service.

## Example RTTTL Usage
The AZTouch board includes a buzzer that can play melodies. Example configuration:
```yaml
service: esphome.termostato_meteo_play_rtttl
data:
  song_str: "siren:d=8,o=5,b=100:d,e,d,e,d,e,d,e"
```
![hass](https://github.com/niahane/meteo-thermostat/blob/7e52d860cf970f4f9c97ee505d01e0b927ff10db/readme_img/hass_thermostat.jpg)
A demostation video:
https://github.com/niahane/meteo-thermostat/blob/7a2923ba1b6a4c6382c3172988ddbd63d4416278/readme_img/video.mp4?raw=true
## Feedback and Suggestions
This project has been a great learning experience, and I hope it proves useful in your setups. If you have any feedback, feature requests, or issues, feel free to reach out via the GitHub issue tracker or at meconiotech@gmail.com.

Enjoy your smart thermostat!

