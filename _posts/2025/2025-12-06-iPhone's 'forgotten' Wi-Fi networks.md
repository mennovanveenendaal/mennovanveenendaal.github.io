---
title: "iPhone's \"forgotten\" Wi-Fi networks"
layout: post
categories: [iOS, wifi, plist]
description: Do you remember that Wi-Fi network you removed from your iPhone's Known Networks? Your iPhone does.
image:
  path: /assets/2025/iphone_wifi_3/forgotten_wifi.png
  alt: iPhone wifi
---

When investigation the [scanning of Wi-Fi networks](https://www.mennovanveenendaal.com/posts/iPhone-Wi-Fi-Scanning/) by an iPhone I acquired a Partially Restored Filesystem Backup to search for traces of scanned Wi-Fi networks. While searching the data for the SSID of a test network, I found a reference in a file named _removed networks_. To find out more about this file, I examined it further.

## Logging
With [UFADE](https://github.com/prosch88/UFADE) I created another Partially Restored Filesystem Backup (PRFS) from the iPhone 12 running iOS 26.1. From this backup I extracted the file *com.apple.wifi.removed-networks.plist* from `\private\var\mobile\Library\Preferences\`. Inside this [.plist](https://en.wikipedia.org/wiki/Property_list) I found several SSIDs of networks I had previously connected my iPhone to and later removed from the Known Networks of the iPhone.

## Script
Using the iOS_sysdiagnose_forensic_scripts by [cheeky4n6monkey](https://github.com/cheeky4n6monkey/iOS_sysdiagnose_forensic_scripts/blob/master/sysdiagnose-net-ext-cache.py) as reference, I created a Python script to parse the *com.apple.wifi.removed-networks.plist* file:

```python
#! /usr/bin/env python
import sys
from optparse import OptionParser
import plistlib


if sys.version_info[0] < 3:
    print("Must be using Python 3! Exiting.")
    exit(-1)

usage = "\n%prog -i inputfile\n"

parser = OptionParser(usage=usage)
parser.add_option("-i", dest="inputfile", 
                  action="store", type="string",
                  help="\private\var\mobile\Library\Preferences\com.apple.wifi.removed-networks.plist")             
(options, args) = parser.parse_args()

if len(sys.argv) == 1:
    parser.print_help()
    exit(-1)

with open(options.inputfile, 'rb', ) as fp:
    pl = plistlib.load(fp, dict_type=dict)
    if pl.items():
        for key, list1 in pl.items():
            print("==================================")
            print(key)
            for item, list1 in pl[key].items():
                if item == "CaptiveProfile":
                    print("CaptiveProfile:")
                    for item2 in list1:
                        print("  - " + item2 + ": " + str(list1[item2]))
                else:
                    print(item + " : " + str(list1))
```
```shell
> python3 removed-wifi-plist.py -i com.apple.wifi.removed-networks.plist

=============================
wifi.network.ssid.TestingNetwork
SSID : b'TestingNetwork'
SupportedSecurityTypes : WPA2 Personal
CaptiveProfile:
  - CaptiveNetwork: False
RemovedAt : 2025-09-26 09:49:58.266271
=============================
wifi.network.ssid.ANewNetwork
RemovedAt : 2025-09-29 10:50:31.770377
SupportedSecurityTypes : WPA2 Personal
CaptiveProfile:
  - CaptiveNetwork: False
SSID : b'ANewNetwork'
=============================
wifi.network.ssid.SomeHotelWiFi
SSID : b'SomeHotelWiFi'
SupportedSecurityTypes : OWE Transition
RemovedAt : 2025-09-19 08:52:01.437784
CaptiveProfile:
  - WhitelistedCaptiveNetworkProbeDate: 2022-12-01 10:11:54.910794
  - CaptiveNetwork: False
  - IsWhitelistedCaptiveNetwork: False
  - NetworkWasCaptive: True
  - CaptiveWebSheetLoginDate: 2022-11-21 16:39:56.171760
=============================
[...]
```

The networks and timestamps match networks the iPhone was once connected to.

### Testing
To confirm this, I removed several other Wi Fi networks, rebooted the iPhone and created a new Partially Restored Filesystem Backup and parsed the file again.

```shell
> python3 removed-wifi-plist.py -i com.apple.wifi.removed-networks.plist

[...]
=============================
wifi.network.ssid.Ziggo1234567890
SSID : b'Ziggo1234567890'
SupportedSecurityTypes : WPA Personal
RemovedAt : 2025-12-03 16:44:52.159261
=============================
wifi.network.ssid.Restaurant_Guest
SSID : b'Restaurant_Guest'
SupportedSecurityTypes : WPA2 Personal
CaptiveProfile:
  - CaptiveNetwork: False
RemovedAt : 2025-12-01 18:18:35.246726
=============================
[...]
```

The timestamps are listed in UTC and match the moments I removed these networks.

Finally, I removed another Wi Fi network and created a new Partially Restored Filesystem Backup almost immediately, without rebooting the device. The network appeared in the plist:

```shell
> python3 removed-wifi-plist.py -i com.apple.wifi.removed-networks.plist

[...]
=============================
wifi.network.ssid.Ziggo92742
SSID : b'Ziggo92742'
SupportedSecurityTypes : WPA2 Personal
RemovedAt : 2025-12-05 18:12:49.985504
=============================
```

## Conclusion
This plist is a valuable source for identifying Wi Fi networks that have been removed. Even when a network is deleted from Known Networks, the SSID remains stored in this file. While I can confirm the timestamps listed in the [testing](#testing) paragraph, it is unclear how long this plist retains entries or whether the timestamps remain accurate over longer periods. I am almost certain that some networks were removed earlier than the dates currently listed, such as the `SomeHotelWiFi` entry. Further testing is needed to confirm this. 

