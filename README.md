# ESPHome Aranet4

This ESPHome package will read data from an Aranet4 CO2 sensor and expose it as Home Assistant sensors. I use a [ESPHome Bluetooth Proxy](https://esphome.github.io/bluetooth-proxies/) with a ESP32 because it is easy to set up, but any other Bluetooth-enabled ESPHome device will also work.

I developed this for the Aranet4 Home, but it should also work with the Pro model. In its current form, only one Aranet4 sensor is supported per ESPHome device.

## Requirements

For this to work, the Aranet4 needs

* to have the firmware version v1.2.0 or greater and
* Smart Home integration must be enabled.

Both of these things can be checked in the Aranet4 Android/iOS mobile app.

## Usage

First, you have to figure out the MAC address of the Aranet4 device. You can do this by pairing your phone with the Aranet4 and checking the MAC address in your phone's settings, or use a BLE scanner app.

Then, add the follow configuration to your ESPHome:

```yaml
substitutions:
  aranet4_name: "Aranet4 Home"
  aranet4_mac_address: "XX:XX:XX:XX:XX:XX"
packages:
  stefanthoss.esphome-aranet4: github://stefanthoss/esphome-aranet4/esphome-aranet4.yaml@main
```

The `aranet4_name` parameter is used to name the sensor in Home Assistant. `aranet4_mac_address` should be the MAC address of the Aranet4.

## Home Assistant Sensors

When set up correctly, the following sensors will be created within the ESPHome integration:

* `CO2`: CO2 concentration in ppm
* `Temperature`: Temperature in °C (will be converted to °F if your Home Assistant is configured that way)
* `Humidity`: Humidity in %
* `Pressure`: Air pressure in hPa
* `Battery`: Battery level in %
* `Status`: CO2 status (1 -> Good, 2 -> Average, 3 -> Unhealthy)
* `Interval`: Measurement interval in seconds (can be configured in the app)
* `Ago`: Time since last measurement in seconds

## Mapping Status in Home Assistant

You can create a template sensor that maps the `Status` sensor's integer value to readable text:

```yaml
template:
  - sensor:
      - name: "Aranet4 Home Indicator"
        state_class: "measurement"
        state: >
          {% if states('sensor.aranet4_home_status') | int == 1 %}
            Good
          {% elif states('sensor.aranet4_home_status') | int == 2 %}
            Average
          {% elif states('sensor.aranet4_home_status') | int == 3 %}
            Unhealthy
          {% endif %}
```

## Source

I have most of my knowledge about how to parse the Aranet4 Bluetooth data from the [Aranet4-Python](https://github.com/Anrijs/Aranet4-Python) project, specifically the [lines of code](https://github.com/Anrijs/Aranet4-Python/blob/4fe309261376cb73417074f25eb1daefb525ebd3/aranet4/client.py#L317-L333) that decode the 13-Byte vector that Aranet4 returns.
