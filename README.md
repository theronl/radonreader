# RadonReader 2022 RD200 v2 (=>2022)

This project provides a tool which allows users collect current radon data from FTLab Radon Eye RD200 v1 and V2 (2022+) (Bluetooth only versions). 

EtoTen v0.4 - 07/05/2022
- Forked Project
- Changed compatability to Python3 
- Added support for new RD200 models made in 2022
- Added auto-scan ability 
- Change the read function to call the handler directly, instead of interacting with the UUIDs

Note: if specifying an (-a) MAC address, you now also have to specify a device type (-t) (either 0 for original RD200 or 1 for RD200 v2)

# Pre-req install steps:

<pre><code>sudo apt install libglib2.0-dev
pip3 install bluepy
pip3 install paho-mqtt
sudo setcap cap_net_raw+e /home/pi/.local/lib/python3.7/site-packages/bluepy/bluepy-helper
sudo setcap cap_net_admin+eip /home/pi/.local/lib/python3.7/site-packages/bluepy/bluepy-helper
</pre></code>

# Pre-req install steps if you're using pyenv:

<pre><code>sudo apt install libglib2.0-dev
pip install bluepy paho-mqtt
sudo setcap cap_net_raw+e "$(find `pyenv prefix` -name bluepy-helper)"
sudo setcap cap_net_admin+eip "$(find `pyenv prefix` -name bluepy-helper)"
</pre></code>

# Home assistant integration via MQTT:

- Install mosquitto MQTT add-on in HA, configure it with a local user and password
- Install mosquitto MQTT integration in HA
- On the host machine run: <pre><code>python3 radon_reader.py -v -ms localhost -mu radonuser -mw radon123  -ma -m</pre></code>  and listen to:
 "environment/RADONEYE/#" in the MQTT integration in HA to verify messages are being sent
 
- Now make the app send automatic updates to HA every 3 minutes
 <pre><code>crontab -e</pre></code> 
 <pre><code>*/3 * * * * /usr/bin/python3 /home/pi/radonreader/radon_reader.py -v -a 94:3c:c6:dd:42:ce -t 1 -ms localhost -mu radonuser -mw radon123  -mw radon123  -ma -m #update radon reading via MQTT every 3 minutes</pre></code> 

- Add this to configuration.yaml:
<pre><code>
mqtt:
  sensor:
    - state_topic: "environment/RADONEYE/#"
      name: 'Radon Level'
      unique_id: 'radon_level'
      unit_of_measurement: 'pCi/L'
      value_template: "{{ value_json.radonvalue }}"
</pre></code>

- Now you can add a sensor card to your HA view

# Hardware Requirements
- FTLabs RadonEye RD200 v1 or v2
- Raspberry Pi w/Bluetooth LE (Low Energy) support (RPi 3B/4/etc...)

# Software Requirements
- Python 3.7
- bluepy Python library
- paho-mqtt Python library

# History
- 0.4 - Forked and modified extensively 
- 0.3 - Added MQTT support

# Usage
<pre><code>usage: radon_reader.py [-h] [-a] ADDRESS [-t] DEVICE_TYPE [-b] [-v] [-s] [-m] [-ms MQTT_SRV]
                       [-mp MQTT_PORT] [-mu MQTT_USER] [-mw MQTT_PW] [-ma]

RadonEye RD200 (Bluetooth/BLE) Reader

optional arguments:
  -h, --help       show this help message and exit
  -a ADDRESS       Bluetooth Address (AA:BB:CC:DD:EE:FF format)
  -t TYPE          0 for original RD200, 1 for RD200 v2 (=>2022)
  -b, --becquerel  Display radon value in Becquerel (Bq/m^3) unit
  -v, --verbose    Verbose mode
  -s, --silent     Only output radon value (without unit and timestamp)
  -m, --mqtt       Enable send output to MQTT server
  -ms MQTT_SRV     MQTT server URL or IP address
  -mp MQTT_PORT    MQTT server service port (Default: 1883)
  -mu MQTT_USER    MQTT server username
  -mw MQTT_PW      MQTT server password
  -ma              Enable Home Assistant MQTT output (Default: EmonCMS)</code></pre>

# Example usage:
<pre><code>python3 radon_reader.py -a 94:3c:c6:dd:42:ce -t 1 -v #verbose output/ specific device MAC
python3 radon_reader.py -v #verbose output, auto scan
python3 radon_reader.py -v -a 94:3c:c6:dd:42:ce -t 1 -ms localhost -mu radonuser -mw radon123  -ma -m #verbose output, specific device MAC, mqtt to home assistant
</pre></code>
