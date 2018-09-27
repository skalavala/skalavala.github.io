---
layout: post
title:  "Using Bluetooth on your RPi3 as a Device Tracker"
author: skalavala
categories: [ bluetooth, RaspberryPi, device_tracker, homeassistant, mqtt, presence-detection ]
image: assets/images/17.jpg
---
Here is a simple way to use your Raspberry Pi's bluetooth for presence detection. You need Raspberry Pi3 in order to be able to leverage bluetooth capabilities. 

To get started, make sure you have python `bluetooth` libraries installed by running the following commands. Also make sure the Bluetooth is enabled on the RPi.

```
sudo apt-get install bluetooth
sudo apt-get install python-bluez
sudo apt-get install bluetooth libbluetooth-dev
sudo python3 -m pip install pybluez
```

After running the above commands one after another, you should be able to run `python3` command line and be able to inport `bluetooth` package.

The following is the python program that checks for statuc change every 15 seconds, and if there is any change, it publishes the message to the MQTT topic.

Have an MQTT `binary_sensor` in Home Assistant and use this as just another data set for your device trackers.

```
#!/usr/bin/python
import bluetooth
from time import sleep
import paho.mqtt.client as mqtt
import paho.mqtt.publish as publish

phones = [
    {'name': 'person1', 'state': 'not home', 'mac': '<mac address>'}
    {'name': 'person2', 'state': 'not home', 'mac': '<mac address>'}
]

client = mqtt.Client("hass-client")
client.tls_set('/home/pi/certs/ca-certificates.crt')
client.username_pw_set('<username>', '<password>')
client.connect('<mqtt broker>', 8883)
client.loop_start()

while True:
    for phone in phones:
        key = "bluetooth/presence/" + phone['name']
        result = bluetooth.lookup_name(phone['mac'], timeout=3)
        if result != None:
            detected_state = 'home'
        else:
            detected_state = 'not home'

        if phone['state'] != detected_state:
            phone['state'] = detected_state
            client.publish(key, detected_state, retain=False)
    sleep(15)

client.disconnect()
```
Thanks to <a target="_blank" href="https://github.com/b10m">@B10m</a> for providing this tip!
