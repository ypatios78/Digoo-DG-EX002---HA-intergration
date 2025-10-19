Digoo DG-EX002 Weather Station - 433MHz Integration
The Digoo DG-EX002 outdoor sensor successfully decodes using the inFactory-TH protocol (compatible with FreeTec NC-3982/Nexus protocol family).

Hardware Requirements
ESP32 (WROOM 32D or compatible)
CC1101 433MHz transceiver module
50mm antenna wire (quarter-wave for 433MHz)
Wiring CC1101 to ESP32
CC1101 Pin	ESP32 Pin
GND	GND
VCC	3.3V
GDO0	GPIO 12
CSN	GPIO 5
SCK	GPIO 18
MOSI	GPIO 23
MISO	GPIO 19
GDO2	GPIO 27 (optional)
ANT	50mm wire antenna
Software Setup
Option 1: OpenMQTTGateway (Recommended)
Flash esp32dev-rtl_433 firmware from: https://docs.openmqttgateway.com/upload/web-install.html
Configure WiFi and MQTT credentials
Sensor will auto-discover in Home Assistant
Option 2: rtl_433_ESP Library
Use the rtl_433_ESP library with Arduino IDE - includes 200+ protocol decoders.

Decoded Protocol Details
Protocol: inFactory-TH (Nexus-compatible)

Example Output:
{
"model": "inFactory-TH",
"id": 84,
"channel": 1,
"battery_ok": 1,
"temperature_C": 24.78,
"humidity": 53,
"mic": "CRC"
}

text

Transmission Details
Frequency: 433.92 MHz
Modulation: OOK (On-Off Keying)
Transmission Interval: ~30-60 seconds
Battery: 2x AAA
Signal Strength: Excellent at close range (-23 dBm at 10cm)
Home Assistant Integration
Sensors auto-discover via MQTT as:

sensor.infactory_th_[channel]_[id]_temperature
sensor.infactory_th_[channel]_[id]_humidity
sensor.infactory_th_[channel]_[id]_battery
Troubleshooting
No signals detected:

Ensure 17.4cm antenna is connected to CC1101
Place sensor within 2-3 meters for testing
Check GPIO pin configuration matches wiring
Verify fresh batteries in sensor
Wrong GPIO configuration:
commands/MQTTtoRTL_433/config {"gdo0":4,"gdo2":-1}

text

Compatible Sensors
The inFactory-TH protocol also decodes:

FreeTec NC-3982-913
Nexus protocol family sensors
Various Chinese 433MHz weather sensors
Resources
OpenMQTTGateway: https://docs.openmqttgateway.com/
rtl_433 Protocol List: https://github.com/merbanan/rtl_433
ESP32 Pinout: https://randomnerdtutorials.com/esp32-pinout-reference-gpios/
Credits
Decoded with rtl_433_ESP library and OpenMQTTGateway v1.8.1

Important for home assistant every time you remove batteries mqtt creates new device

Stop Creating New Devices

Turn OFF auto-discovery temporarily:

In HA, find OMG_rtl_433_ESP device

Turn OFF switch: "SYS: Auto discovery"​

Create Manual MQTT Sensor also

Add this to your configuration.yaml:

mqtt:
sensor:
- name: "Outdoor Temperature"
  state_topic: "home/OMG_rtl_433_ESP/RTL_433toMQTT/inFactory-TH/1/+"
  value_template: "{{ value_json.temperature_C | round(1) }}"
  unit_of_measurement: "°C"
  device_class: temperature
  state_class: measurement

- name: "Outdoor Humidity"
  state_topic: "home/OMG_rtl_433_ESP/RTL_433toMQTT/inFactory-TH/1/+"
  value_template: "{{ value_json.humidity | int }}"
  unit_of_measurement: "%"
  device_class: humidity
  state_class: measurement
  
- name: "Outdoor Sensor Battery"
  state_topic: "home/OMG_rtl_433_ESP/RTL_433toMQTT/inFactory-TH/1/+"
  value_template: "{{ 'OK' if value_json.battery_ok == 1 else 'Low' }}"
  device_class: battery
  
- name: "Outdoor Sensor RSSI"
  state_topic: "home/OMG_rtl_433_ESP/RTL_433toMQTT/inFactory-TH/1/+"
  value_template: "{{ value_json.rssi | int }}"
  unit_of_measurement: "dBm"
  device_class: signal_strength
  state_class: measurement
