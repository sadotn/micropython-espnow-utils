# Micropython ESPNow Utilities
A collection of small utilities for use with espnow and wifi on micropython

- `src/wifi.py`: A small module to bulletproof initialising wifi on ESP32 and
  ESP8266 devices.

These functions are designed to robustly and reliably account for differences
between esp8266 and esp32 devices and to ensure the wifi is always in a fully
known state (even after soft_reset).

```python
import wifi

sta, ap = wifi.reset()   # STA_IF on and disconnected, AP off, channel=1
sta, ap = wifi.reset(sta=True, ap=True, channel=6)
wifi.connect("ssid", "password")
wifi.status()            # Print details on wifi config
```

- `src/scan_for_peer.py`: Scan channels to find an ESPNow peer.

```python
from scan_for_peer import scan_for_peer

scan_for_peer(b'macadd')  # Print channel if found and leave set to channel

```

- `src/lazyespnow.py`: LazyESPNow() extends ESPNow class to catch and fix most
  common errors.

```python
from lazyespnow import LazyESPNow

peer = b'\xaa\xaa\xaa\xaa\xaa\xaa'
enow = LazyESPNow()
enow.send(peer, message)    # Automatically call active(True) and add_peer()
```

- `src/ntp.py`: Set the time from an NTP server over wifi.

```python
import ntp
ntp.server = "192.168.1.1"
ntp.settime()
```

- `src/timer.py`: A useful collection of easy to user python timer objects.

```python
import timer

for t in timer_ms(5000, 1000, raise_on_timeout=True):
    print(t)
```

- `src/echo.py`: A simple ESPNow server that echos messages back to the sender.

```python
import echo

echo.server()       # Wait for incoming messages and echo back to sender
```

```python
import espnow
import network
import time
import ubinascii

import components.device_type.DeviceType as DeviceType

class ESPNowController():

    def __init__(self):
        self.wlan = network.WLAN(network.STA_IF)  # network.STA_IF Or network.AP_IF
        self.wlan.active(True)        
        print ('wifi initialized')
        self.esp_now = espnow.ESPNow()
        self.esp_now.init()
        print ('esp_now initialized')

    def get_current_mac(self):
        return ubinascii.hexlify(self.wlan.config('mac'),':').decode()
        # return self.wlan.config('mac')

    def add_peer(self, mac):
        self.esp_now.add_peer(mac)

    def send_data(self, data):
        try:
            self.esp_now.send("test")
        except OSError as err:
            print("Error: {0}".format(err))

        time.sleep_ms(1000)    
```
