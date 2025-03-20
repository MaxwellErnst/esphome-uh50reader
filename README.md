# esphome-uh50reader
ESPHome custom component for communicating with Landis+Gyr T550 (UH50) heat/cold meters and reading usage data. The UH50 meter communicates over an optical interface using the standardized IEC 62056-21 protocol. If the meter is battery powered, each request for data will drain the battery life. 

## ESPHome version
The current version in main is tested with ESPHome version `2025.2.2`. Make sure your ESPHome version is up to date if you experience compile problems.

## Hardware
The optical eye hardware I'm using was ordered from here https://www.aliexpress.com/item/1005005206147556.html?spm=a2g0o.productlist.main.19.518a609aIFHYW8&algo_pvid=0e6afd1c-6482-4cc7-91d2-8b25f4c646bd&algo_exp_id=0e6afd1c-6482-4cc7-91d2-8b25f4c646bd-9&pdp_ext_f=%7B%22order%22%3A%226%22%2C%22eval%22%3A%221%22%7D&pdp_npi=4%40dis%21SEK%21274.82%21274.82%21%21%21192.36%21192.36%21%40211b80d117419017804413989e593e%2112000032158161996%21sea%21SE%210%21ABX&curPageLogUid=69DZwhRRr4yO&utparam-url=scene%3Asearch%7Cquery_from%3A
TTL IR infra-red light IEC1107 DLMS bi-directional communication kWh meter electricity and gas tariff meters TTL Optical Probe

This optical eye is then connected to a NodeMCU ESP-controller with the RX pin connected to the RX pin (GPIO3) on the NodeMCU and the TX pin connected to the D4 pin (GPI17) on the ESP32.

### UART
Since the protocol requires outgoing communication (TX) using 300 baud and the incoming communication (RX) using 2400 baud, both hardware UARTs on the ESP32 are used. ESP8266 based controllers wont work with this buffersize. 

### Parts
- 1 Optical eye, see recommendation above
- 1 ESP32 
- 1 Wires
- Wall plug and USB cable to power the NodeMCU

## Installation
Clone the repository and create a companion `secrets.yaml` file with the following fields:
```
wifi_ssid: <your wifi SSID>
wifi_password: <your wifi password>
fallback_password: <fallback AP password>
hass_api_password: <the Home Assistant API password>
ota_password: <The OTA password>
encryption_key: <Your Encryption Key>
```
Make sure to place the `secrets.yaml` file in the root path of the cloned project. The `fallback_password` and `ota_password` fields can be set to any password before doing the initial upload of the firmware.

Prepare the microcontroller with ESPHome before you connect it to the circuit:
- Install the `esphome` [command line tool](https://esphome.io/guides/getting_started_command_line.html)
- Plug in the microcontroller to your USB port and run `esphome uh50reader.yaml run` to flash the firmware
- Remove the USB connection and connect the microcontroller to the rest of the circuit and plug it into the P1 port.
- If everything works, your Home Assistant will now auto detect your new ESPHome integration.

You can check the logs by issuing `esphome uh50reader.yaml logs` (or use the super awesome ESPHome dashboard available as a Hass.io add-on or standalone). 
```
[01:09:42][I][cmd:097]: data cmd sent
[01:09:42][D][sensor:113]: 'Cumulative Active Import': Sending state 85548.00000 kWh with 2 decimals of accuracy
[01:09:42][D][sensor:113]: 'Cumulative Volume': Sending state 2144.70996 m3 with 6 decimals of accuracy
```

## Home Assistant service to call when to make a reading

This version exposes a home assistant service that you have to call either manually or through an automation when to read the meter.

## Build and install tips for Windows

Use the esphome docker image to build it without having to install the build tools.
```docker run -p 6052:6052 -v C:\some-directory\esphome-uh50reader:/config -it esphome/esphome```
Open a web browser to http://localhost:6052
Click install and download the firmware using modern format, then flash it

## Technical documentation
UH50 overview:
https://www.landisgyr.com/webfoo/wp-content/uploads/product-files/ConfigInstr_m_T550_UH50_en.pdf

IEC 62056 telegram structure:
http://manuals.lian98.biz/doc.en/html/u_iec62056_struct.htm
