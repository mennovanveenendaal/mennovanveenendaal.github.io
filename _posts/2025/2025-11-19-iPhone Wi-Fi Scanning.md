---
title: "iPhone Wi-Fi Scanning"
layout: post
categories: [iOS, wifi]
description: iPhone is silently recording nearby Wi-Fi information
image:
  path: /assets/2025/iPhone_wifi_2/wifi.png
  alt: iPhone wifi
---

To connect to a WiFi network, an iPhone needs to scan its surroundings to locate available networks. It must then process the detected SSID to determine whether it should connect. This raised a question for me. What happens to all the SSID values that are scanned but never used, especially when an iPhone collects a significant amount of [data](https://www.mennovanveenendaal.com/posts/The-Mystery-of-iPhone-Wi-Fi-Sync/#apple-unified-logs)? Could the device store logging of SSID values it has only observed but not connected to?

## WiFi
To find out, I created a new Wi-Fi network named *NotConnectedAtAll*, configured on channel 13 with wep security. Since no other network within range used this channel or security protocol, this setup made the test network easier to identify in the logging.

![SSID](/assets/2025/iPhone_wifi_2/NotConnectedAtAll.png)
_Fig.1 SSID_
 
## Testing
My initial test used an iPhone 5s running iOS 12.5.7. Because this model has a small internal memory, extracting data was faster and better suited for the first experiment.

## Logging
Using [UFADE](https://github.com/prosch88/UFADE) I could not capture the Apple Unified Logs, because iOS 12.5.7 is too old for this feature. Instead, I captured the livelog.
In the logfile I found a WiFi network on channel 13 with an [RSSI](https://en.wikipedia.org/wiki/Received_signal_strength_indicator) value that indicated it was close to the device. The SSID was redacted as `<private>`:
```
SyslogEntry(pid=55, timestamp=datetime.datetime(2025, 11, 4, 7, 25, 1, 167665), level=<SyslogLogLevel.DEBUG: 2>, image_name='/usr/libexec/locationd', filename='/usr/libexec/locationd', message='WifiScan, addAp, 6, insert, <private>, ssid, <private>, rssi, -28, channel, 13, age, 2.5, timestamp, 783934378.7', label=SyslogLabel(category='GeneralCLX', subsystem='com.apple.locationd.Position'))
```

After that I created and extracted a sysdiagnose using UFADE. In the `/sysdiagnose/sysdiagnose.log` files I found multiple references to the WiFi logging process:
```
2025-11-04 08:31:50: Tar streaming from: /var/mobile/Library/Logs/CrashReporter/DiagnosticLogs/sysdiagnose/IN_PROGRESS_sysdiagnose_2025.11.04_08-31-30+0100_iPhone_OS_iPhone_16H81.tmp/WiFi/wifi-buf-11-04-2025__08:31:46.935.log.tgz, to: IN_PROGRESS_sysdiagnose_2025.11.04_08-31-30+0100_iPhone_OS_iPhone_16H81/WiFi/wifi-buf-11-04-2025__08:31:46.935.log.tgz
```

### Wi-Fi buffer
Following this path I found multiple wifi-buf log files in the Wi-Fi folder. These contained sequences showing the results of Wi-Fi scanning.

![Wifi_folder](/assets/2025/iPhone_wifi_2/sysdiagnose_wifi.png)
_Fig.2 Wifi folder_

In the logfile for 4 November I found entries showing that the broadcast scan detected the *NotConnectedAtAll* SSID:

```
11/04/2025  8:25:41.921 Scanning For Broadcast found: [...], , , [...], [...], , , , [...], [...], NotConnectedAtAll, , 
11/04/2025  8:25:41.921 Auto join scan found: 
11/04/2025  8:25:41.921 __WiFiDeviceManagerPrepareCandidates: current state: Scanning For Broadcast 
11/04/2025  8:25:41.921 Preparing GAS queries for HS2.0 networks
11/04/2025  8:25:41.921 No GAS queries.
11/04/2025  8:25:41.921 __WiFiDeviceManagerPrepareCandidates: current state: Scanning For Broadcast 
11/04/2025  8:25:41.922 __WiFiDeviceManagerPrepareCandidates: no scan candidate network
11/04/2025  8:25:41.922 __WiFiDeviceManagerDetermineNextAction: current state: Scanning For Broadcast
11/04/2025  8:25:41.922 ATJManager: atjEnabled=0 userInteractionMode=NonInteractive discovered 13 networks
```

### Historical
The other wifi-buf logs contained earlier scanning information. The oldest logfile was dated 26 September 2025, while the sysdiagnose was created on 4 November. SSID values scanned on earlier dates were also recorded.

```
 9/26/2025 17:17:17.567 Auto join scan completed (0) with current state: Scanning For Broadcast
 9/26/2025 17:17:17.567 __WiFiDeviceManagerStateMachineRun: current state: Scanning For Broadcast
 9/26/2025 17:17:17.568 Scanning For Broadcast found: [...], [...], [...], , [...], , , [...], , , [...], [...]
 9/26/2025 17:17:17.568 Auto join scan found: 
 9/26/2025 17:17:17.568 __WiFiDeviceManagerPrepareCandidates: current state: Scanning For Broadcast 
 9/26/2025 17:17:17.568 Preparing GAS queries for HS2.0 networks
 9/26/2025 17:17:17.568 No GAS queries.
 9/26/2025 17:17:17.568 __WiFiDeviceManagerPrepareCandidates: current state: Scanning For Broadcast 
 9/26/2025 17:17:17.568 __WiFiDeviceManagerPrepareCandidates: no scan candidate network
 9/26/2025 17:17:17.568 __WiFiDeviceManagerDetermineNextAction: current state: Scanning For Broadcast
 9/26/2025 17:17:17.568 ATJManager: atjEnabled=0 userInteractionMode=Interactive discovered 12 networks
```

## Test Setup iPhone 12
For the second test I used an iPhone 12 on iOS 26.0.1. I disconnected it from all WiFi networks and placed it within range of the access point broadcasting *NotConnectedAtAll*.

### Logs
In the Apple Unified Logs I found entries from `com.apple.WiFiManager` that matched the test network:
```
2025-11-04 17:25:00.197191+0100 0x48511    Default     0x0                  54     0    wifid: (CoreWiFi) [com.apple.WiFiManager:] [corewifi] AUTO-JOIN: -- <redacted> - ssid=<redacted> (1956089320), bssid=<redacted>, security=wep, channel=2g13/40, cc=NL, phy=n, rssi=-8, wasConnectedDuringSleep=0, bi=100, age=1345ms (33676966972125)
```
The channel and security type matched the test setup. The RSSI suggested it was in close proximity. 

After extracting the sysdiagnose I did not find any wifi-buf log files. Searching for references to them in the logarchive showed entries, but no actual wifi-buf logs.
```
$ grep -r "wifi-buf-"
Binary file system_logs.logarchive/dsc/CC78AB9B8F8634F5BC969EAE017BAACE matches
Binary file system_logs.logarchive/73/09EFC78FD63DC8ADA6E2AF781E382D matches
```

```
MM/dd/yyyy HH:mm:ss.SSS com.apple.wfloggercircularbuffer com.apple.WiFiPolicy.WFLogger.dump /var/mobile/Library/Logs/CrashReporter/WiFi MM-dd-yyyy__HH:mm:ss.SSS /var/mobile/Library/Logs/CrashReporter/WiFi/wifi-buf-%d-%@.log wb com.apple.wifi.trafficEngineering com.apple.wifi.trafficeng.event Enter %s -[WFTrafficEngManager initWithTrafficEngDelegate:] %s self alloc failed %s Dispatch Queue Creation Failed %s Invalid trafficEngDelegate Leave %s 
[...]
```

```
[...]
IO80211Adaptive11rOverride /var/mobile/Library/Logs/CrashReporter/WiFi/WiFiManager wifi-buf- .log Error removing %@: %@ com.apple.WiFiManager /var/mobile/Library/Logs/CrashReporter/WiFi wifimanager /Library/Logs/wifimanager.log %s: wifimanager is NULL __WiFiLoggingTurnOnWiFiLogging __WiFiLoggingTurnOffWiFiLogging WiFiChipResetCompleted -[WiFiFindAndJoinRequest initWithNetworkName:] < %@:%p  Channel=%d  Band=%d  Timeout=%d  -[WiFiFindAndJoinRequest _canPerformRetry:] 
[...]
```

### CRC-32 Checksum
In the iPhone 12 logs I repeatedly saw the same numeric value after the redacted SSID, 1956089320. Searching for this value returned several files, including files with names that suggested Wi-Fi scans.

```
user@DESKTOP: /sysdiagnose_2025.11.17_14-46-27+0100_iPhone-OS_iPhone_23A355$ grep -r "
1956089320"

Binary file system_logs.logarchive/logdata.LiveData.tracev3 matches
Binary file system_logs.logarchive/Persist/0000000000009cfa.tracev3 matches
Binary file system_logs.logarchive/Persist/0000000000009cfb.tracev3 matches
Binary file system_logs.logarchive/Persist/0000000000009cfc.tracev3 matches
WiFi/wifi_scan.txt:<redacted> - ssid=<redacted> (1956089320), bssid=<redacted>, security=wep, channel=2g13/40, cc=NL, phy=n, rssi=-10, wasConnectedDuringSleep=0, bi=100, age=1848ms (23857183598375)
WiFi/wifi_scan_cache.txt:<redacted> - ssid=<redacted> (1956089320), bssid=<redacted>, security=wep, channel=2g13/40, cc=NL, phy=n, rssi=-7, wasConnectedDuringSleep=0, bi=100, age=1610ms (23855441334333)
```

The numeric ID appeared to be a hash or checksum. Unable to find what hashing algorithm Apple uses in iOS I asked ChatGPT:

```
The numeric ID is usually derived from a CRC32 or Murmur hash of the SSID[...]
```

ChatGPT was unable to provide any source or proof to support this claim. I tested this using CyberChef with the SSID string *NotConnectedAtAll*, which produced a CRC32 checksum that matched the decimal value 1956089320.

![Cyber Chef](/assets/2025/iPhone_wifi_2/cyberchef.png)
_Fig.3 Cyber Chef output_

This was confirmed using [crc32.online](https://crc32.online/):

![Cyber Chef](/assets/2025/iPhone_wifi_2/decimal_1.png)
_Fig.4 crc32 checksum_

And also by a simple Python script:

```python
import zlib

ssid = "NotConnectedAtAll"

ssid_bytes = ssid.encode('utf-8')
numeric_id = zlib.crc32(ssid_bytes) & 0xFFFFFFFF

print(f"CRC32: {numeric_id}")
```
- source: https://bugs.python.org/msg79542

```
$ python3 crc32.py
CRC32: 1956089320
```

Any other inputs gave a different checksum. The CRC32 value matched the one found in the logging.

### wifi-buf
I could not confirm whether the iPhone 12 on iOS 26.0.1 also produced wifi-buf log files. With my current setup I cannot extract the full filesystem and therefore cannot access `/var/mobile/Library/Logs/CrashReporter/WiFi/`.  
Logical backups, PRFS backups, sysdiagnose captures and crash reports did not include these files.

### Historical data
To determine the first appearance of the `[corewifi] AUTO-JOIN: -- <redacted> - ssid=<redacted>`scan log, I extracted the full Apple Unified Logs on 19 November around 10:30 AM. The earliest entry in the AUL was from 3 November, but the first clear scan result appeared on 18 November at 07:19 AM.

```
2025-11-18 07:19:17.333381+0100 0x4a5f     Default     0x0                  53     0    wifid: (CoreWiFi) [com.apple.WiFiManager:] [corewifi] AUTO-JOIN: Scan SUCCEEDED (duration=144ms, results=20, error=((null)), liveCount=2, cacheOnly=0, maxCacheAge=20000)
[...]
2025-11-18 07:19:17.334248+0100 0x4a5f     Default     0x0                  53     0    wifid: (CoreWiFi) [com.apple.WiFiManager:] [corewifi] AUTO-JOIN: -- <redacted> - ssid=<redacted> (2172880285), bssid=<redacted>, security=wpa2-personal, rsn=[mcast=aes_ccm, bip=none, ucast={ aes_ccm }, auths={ psk }, mfp=no, caps=0x0], channel=2g1/20, cc=BE, phy=n, rssi=-85, wasConnectedDuringSleep=0, bi=100, age=18210ms (1443972220625)
```

The checksum value in that line matched the checksum of a nearby SSID.

![Cyber Chef](/assets/2025/iPhone_wifi_2/cyberchef2.png)
_Fig.5 Cyber Chef output_

## Scanning
When the iPhone was connected to a wireless network, there were no scan results in the logging.

When the iPhone was disconnected from Wi-Fi, I captured a short Live Syslog via UFADE and found scan results including the checksum of the nearby SSID.

```
SyslogEntry(pid=53, timestamp=datetime.datetime(2025, 11, 19, 17, 54, 12, 471149), level=<SyslogLogLevel.NOTICE: 0>, image_name='/System/Library/PrivateFrameworks/CoreWiFi.framework/CoreWiFi', filename='/usr/sbin/wifid', message='[corewifi] AUTO-JOIN: -- <redacted> - ssid=<redacted> (2172880285), bssid=<redacted>, security=wpa2-personal, rsn=[mcast=aes_ccm, bip=none, ucast={ aes_ccm }, auths={ psk }, mfp=no, caps=0xC], channel=2g11/20, cc=EU, phy=n, rssi=-91, wasConnectedDuringSleep=0, bi=200, age=13745ms (27930011187250)', label=SyslogLabel(category='', subsystem='com.apple.WiFiManager'))

```

In a longer Live Syslog captured while the device was connected, no scanning occurred. Instead, the logs focused on connection quality and performance.

## Conclusion
iOS collects extensive data about nearby Wi-Fi networks, including SSID values within range. In the logging these SSIDs are redacted as `<private>`. When the actual SSID is known, the checksum can be calculated and compared with the logged values.

The presence of a checksum confirms that the iPhone was within reach of *a* Wi-Fi network with a matching SSID. However, in the logs available to me I found no traces of MAC addresses for the scanned networks. Therefore, I cannot state with certainty that the iPhone was close enough to a specific access point to verify proximity.

The scan results contain multiple redacted SSIDs and their checksums. In theory an environmental scan at a specific location could reveal active SSIDs, and their checksums could be compared with those in the logs. This could offer an indication of the device’s location at a given moment in time, although without BSSID values it cannot be considered definitive.