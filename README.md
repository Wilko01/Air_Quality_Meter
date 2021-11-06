# Air_Quality_Meter
By using a 3D printer in house it is important to understand the health impact like micro particles which are generated in the build process of the products that the 3D printer makes. The SDS010 is used to measure the air quality.

## Description and operation instructions
The SDS010 monitors the air quality by neasuring the 2.5 and 10 um particles in the air. The SDS010 has a lifetime of 8000 hours, hence the measurement is reduced to periodic measurements. In this case every 10 minutes. The ESP32-WROOM-32D is configured via the ESPHome plugin in Home Assistant. 

 ## Technical description
The SDS010 is connected to the ESP32-WROOM-32D

| SDS101  | ESP32      |
| --------|------------|
| TX      | rx_pin: 16 |
| RX      | tx_pin: 17 |
| GND     | GND        |
| 5V      | 5V         |


### Parts
1 x ESP32-WROOM-32D

<img src="Images/ESP32-WROOM-32D.jpg" alt="drawing" width="150"/>

1 x SDS010

<img src="Images/SDS010.jpg" alt="drawing" width="500"/>

### Schematic overview
<img src="Images/Schematic_overview.jpg" alt="drawing" width="500"/>

### ESPHome installation
See the instructions https://github.com/Wilko01/ESPHome  (not listed here)


### ESPHome Configuration in Home Assistant
Create a new device  with these settings
```
# Defining an SDS011 Air Quality Sensor in ESPHome
esphome:
  name: esp17
  platform: ESP32
  board: esp32doit-devkit-v1

# Enable logging
logger:

# Enable Home Assistant API
api:

ota:
  password: "4764505ce89048d2422149c83dbc51ca"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Esp17 Fallback Hotspot"
    password: "kWzangCTxyzK"

captive_portal:

# Defining the UART
uart:
  rx_pin: 16
  tx_pin: 17
  baud_rate: 9600

sensor:
  - platform: sds011
    pm_2_5:
      name: "Particulates <2.5µm Concentration"
    pm_10_0:
      name: "Particulates <10.0µm Concentration"
    # Set the update interval to balance the accuracy and sensor life
    update_interval: 10min
```

### Interface
#### Home Assistant
Home Assistant is connected via the ESPHome integration. The hazardous values for the 2.5um are different from the values of the 10.0um. Therefore two cards are made in the dashboard.
##### 2.5um card
Create a card in the dashboard of Home Assistant with this code for the 2.5um measurement:
```
type: entities
style:
  .: |
    ha-card {
      --ha-card-box-shadow: none;
      background-color: transparent;
      border-radius: none;

    }
entities:
  - type: custom:mini-graph-card
    entities:
      - entity: sensor.particulates_2_5um_concentration
    icon: mdi:air-filter
    height: 200
    hours_to_show: 96
    more_info: false
    hour24: true
    points_per_hour: 6
    aggregate_func: max
    color_thresholds:
      - value: 0
        color: lightgreen
      - value: 24
        color: green
      - value: 36
        color: yellow
      - value: 48
        color: orange
      - value: 54
        color: red
    line_width: 11
    font_size: 120
    show:
      graph: bar
      average: true
      extrema: true
      name: true
      icon: true
      labels: true
      labels_secondary: true
```

##### 10.0um card
Create a card in the dashboard of Home Assistant with this code for the 10.0um measurement:
```
type: entities
style:
  .: |
    ha-card {
      --ha-card-box-shadow: none;
      background-color: transparent;
      border-radius: none;

    }
entities:
  - type: custom:mini-graph-card
    entities:
      - entity: sensor.particulates_10_0um_concentration
    icon: mdi:air-filter
    height: 200
    hours_to_show: 96
    more_info: false
    hour24: true
    points_per_hour: 6
    aggregate_func: max
    color_thresholds:
      - value: 0
        color: lightgreen
      - value: 34
        color: green
      - value: 51
        color: yellow
      - value: 59
        color: orange
      - value: 76
        color: red
    line_width: 11
    font_size: 120
    show:
      graph: bar
      average: true
      extrema: true
      name: true
      icon: true
      labels: true
      labels_secondary: true

```

### NodeRed
Create a flow to alert when the quality of the air is not in the green area anymore.
```
[{"id":"7c1ab4b.67aba4c","type":"tab","label":"Flow 1","disabled":false,"info":""},{"id":"5170f15d.af318","type":"server-state-changed","z":"7c1ab4b.67aba4c","name":"Air quality 2.5um particles","server":"b64c0ad2.df17f8","version":1,"exposeToHomeAssistant":false,"haConfig":[{"property":"name","value":""},{"property":"icon","value":""}],"entityidfilter":"sensor.particulates_2_5um_concentration","entityidfiltertype":"exact","outputinitially":false,"state_type":"str","haltifstate":"35","halt_if_type":"num","halt_if_compare":"gt","outputs":2,"output_only_on_state_change":false,"for":"","forType":"num","forUnits":"minutes","ignorePrevStateNull":false,"ignorePrevStateUnknown":false,"ignorePrevStateUnavailable":false,"ignoreCurrentStateUnknown":false,"ignoreCurrentStateUnavailable":false,"x":210,"y":380,"wires":[["544cde06.a711d"],[]]},{"id":"544cde06.a711d","type":"function","z":"7c1ab4b.67aba4c","name":"Define email","func":"msg = {\n    payload : \"The air quality is at danger as the amount of 2.5um particles is above 35 particles and colors yelow.\"+ \"\\n\" + \n    \"The amount of 2.5um particles is: \" + msg.payload + \"\\n\" + \n    \"Check the dashboard in Home Assistant\",\n    topic : \"Air quality 2.5um particles alert\",\n};\nreturn msg;\n\n//msg.from = \"HomeAssistant@home.nl\";","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":550,"y":360,"wires":[["40967c8c.140c34"]],"info":"Depending on the incoming message ON or OFF, the check will be executed to turn the light with a certain LUX value based on day or night."},{"id":"40967c8c.140c34","type":"e-mail","z":"7c1ab4b.67aba4c","server":"smtp.ziggo.nl","port":"587","secure":false,"tls":true,"name":"wilko@wmmt.nl","dname":"Mail to Wilko@wmmt.nl","x":810,"y":360,"wires":[]},{"id":"dd2ee594.0b6bd8","type":"server-state-changed","z":"7c1ab4b.67aba4c","name":"Air quality 10.0um particles","server":"b64c0ad2.df17f8","version":1,"exposeToHomeAssistant":false,"haConfig":[{"property":"name","value":""},{"property":"icon","value":""}],"entityidfilter":"sensor.particulates_10_0um_concentration","entityidfiltertype":"exact","outputinitially":false,"state_type":"str","haltifstate":"50","halt_if_type":"num","halt_if_compare":"gt","outputs":2,"output_only_on_state_change":false,"for":"","forType":"num","forUnits":"minutes","ignorePrevStateNull":false,"ignorePrevStateUnknown":false,"ignorePrevStateUnavailable":false,"ignoreCurrentStateUnknown":false,"ignoreCurrentStateUnavailable":false,"x":210,"y":480,"wires":[["52896b9f.10ef64"],[]]},{"id":"52896b9f.10ef64","type":"function","z":"7c1ab4b.67aba4c","name":"Define email","func":"msg = {\n    payload : \"The air quality is at danger as the amount of 10um particles is above 50 particles and colors yelow.\"+ \"\\n\" + \n    \"The amount of 10.0um particles is: \" + msg.payload + \"\\n\" + \n    \"Check the dashboard in Home Assistant\",\n    topic : \"Air quality 10.0um particles alert\",\n};\nreturn msg;\n\n//msg.from = \"HomeAssistant@home.nl\";","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":550,"y":460,"wires":[["c682708.937569"]],"info":"Depending on the incoming message ON or OFF, the check will be executed to turn the light with a certain LUX value based on day or night."},{"id":"c682708.937569","type":"e-mail","z":"7c1ab4b.67aba4c","server":"smtp.ziggo.nl","port":"587","secure":false,"tls":true,"name":"wilko@wmmt.nl","dname":"Mail to Wilko@wmmt.nl","x":810,"y":460,"wires":[]},{"id":"c10ecf40.1e4ad","type":"comment","z":"7c1ab4b.67aba4c","name":"Check air quality by measuring the 2.5 and 10.0 um particles","info":"","x":300,"y":300,"wires":[]},{"id":"b64c0ad2.df17f8","type":"server","name":"Home Assistant","legacy":false,"addon":true,"rejectUnauthorizedCerts":true,"ha_boolean":"y|yes|true|on|home|open","connectionDelay":true,"cacheJson":true}]
```

### Testing
Create a dashboard in Home Assistant and see the measurements come in. Depending on the measurement interval this can take some time.

### Information
- [Rules syntax](https://esphome.io)
- [Air quality sensor with ESPHome](https://cyan-automation.medium.com/creating-an-air-quality-sensor-using-an-sds011-and-esphome-7305f764f6f5)

Generic
- [Markdown Cheat Sheet](https://www.markdownguide.org/cheat-sheet/)


### Problems
..

### Wishlist
..

