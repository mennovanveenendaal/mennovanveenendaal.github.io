---
title: "The Mystery of iPhone Wi-Fi Sync"
layout: post
categories: [iOS, wifi]
image:
  path: /assets/2025/iPhone_wifi/wifi.png
  alt: iPhone wifi
---
Sometimes I notice that my iPhone connects to a Wi-Fi network without me ever setting up that connection or entering a password. Even more surprising, this can happen in places I have never been before, yet my phone somehow knows the Wi-Fi password and connects automatically.

When I checked my list of saved SSIDs, I found several networks I had never connected to. I understand that this information is likely synced through my Apple account, shared between my devices. Still, I find it fascinating how my iPhone can “just know” these details. 

That raised another question: if Wi-Fi settings are synced, are other characteristics, such as connection timestamps, also synchronized? If so, that could make the data misleading, suggesting the device was near a Wi-Fi network it never actually encountered.

## Test Setup
To find out how this syncing works, I used two iPhones, an iPhone 12 (iOS 26.0) and an iPhone 13 (iOS 18.6.2), both signed in to the same Apple account.

For the Wi-Fi I created a new network named "BrandNewNetwork", initially keeping the network disabled.

## Testing
To test synchronization behavior, I turned off the iPhone 12, then enabled the BrandNewNetwork network and connected the iPhone 13 to this Wi-Fi. After a few minutes I disabled the Wi-Fi network again, waited briefly and powered iPhone 12 back on.

When I checked the list of known networks on the iPhone 12, "BrandNewNetwork" was present, including the password!

### Timeline
04 October 2025
- 1405 turnoff iPhone12
- 1406 enabled the Wi-Fi *BrandNewNetwork*
- 1407 connect iPhone13 to wifi *BrandNewNetwork* using password (autojoin option enabled)
- 1410 disabled Wi-Fi *BrandNewNetwork*
- 1411 connected the iPhone13 to a known Wi-Fi network
- 1412 powered the iPhone12 back on, connected to a known Wi-Fi network
- 1414 Check  SSID's in the iPhone's 12 known Wi-Fi list

## SSID in iLEAPP
To see how the Wi-Fi network appeared on the iPhone 12, I generated and captured the sysdiagnose of the iPhone 12 [UFADE](https://github.com/prosch88/UFADE) and processed it with [iLEAPP](https://github.com/abrignoni/iLEAPP). Although iLEAPP does not officially support iOS 26, it successfully parsed the wireless network data.s (amongst other data).

![iLEAPP](/assets/2025/iPhone_wifi/iLEAPP_0410.png)
_Fig.1 iLEAPP WiFi networks_

The *BrandNewNetwork* appeared in the Wi-Fi networks list, showing the "Added at" timestamp (UTC) of the the iPhone 13 connectiong in the [test](#testing)(UTC+2). The *MAC Generation Time* matched the time I powered on the iPhone 12.

I noticed that the basic service set identifier [(_BSSID_)](https://en.wikipedia.org/wiki/Service_set_(802.11_network)) was empty, unlike networks that the iPhone 12 had actually connected to.

### Last Joined Timestamp
The _Last Joined Timestamp_ was empty for all networks, including the one the device was connected to at the time of the extraction. I was unable to determine why these timestamps were missing.

### Link Down Timestamp
The *Link Down Timestamp* appeared only for networks obtained through the shared Apple account, not for those manually added on the device.

## Apple Unified Logs
To understand how the data transfer occurred, I examined the Apple Unified Logs (AUL). Using Tim Korvers [CLI CheatSheet](https://thesisfriday.com/thesis-friday-11-how-to-cli-commands-log/) I was able to acquired the AUL. I collected logs starting from 14:04:00 and filtered them for the time window time window 14:04:00 - 14:15:00. 

```bash
$ log collect --start "2025-10-04 14:04:00" --device-name "iPhone12" --output 
iphone12.logarchive

$ log show --archive iphone12.logarchive --start "2025-10-04 14:04:00" --end "2025-10-04 
14:15:00" > iphone12.log
```

The resulting file contained **677094** events for that 11-minute window. Filtering for “BrandNewNetwork” showed that all Wi-Fi events shared the same PID (54). I then extracted only those entries, using the same time window of 14:04:00 - 14:15:00
```
log show --predicate 'processIdentifier == 54' iphone12.logarchive --start "2025-10-04 14:04:00" --end "2025-10-04 14:15:00" > iphone12_PID54_04102025_140400_141500.log
```

This reduced the logfile to "only" 21358 events. 

## Log Observations
The first relevant event showed that **33** privateMac networks were flushed just before the iPhone 12 was powered off:

```
2025-10-04 14:04:00.545551+0200 0x1de42    Default     0x596d5              54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:]  WFMacRandomisation : WiFiManagerFlushPrivateMacNetworksCache: Successfully flushed 33 privateMac networks to the plist
```

When the iPhone was powered on, it fetched 33 networks from "the cache", updating the "private mac networks".
```
2025-10-04 14:14:27.436824+0200 0x1bc2     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WiFiManagerGetPrivateMacNetworksCache:  WFMacRandomisation : Fetched 33 known networks from the cache
2025-10-04 14:14:27.436864+0200 0x1bc2     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WiFiManagerGetPrivateMacNetworksCache:  WFMacRandomisation : Fetched 33 known networks from the cache
2025-10-04 14:14:27.436919+0200 0x1bc2     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WFMacRandomisation :WiFiManagerAddPrivateMacNetwork:Adding network to private mac cache with reason <11>  existingIndex 32, insertIndex 0
2025-10-04 14:14:27.436922+0200 0x1bc2     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WiFiManagerSetPrivateMacNetworksCache: Updated 33 private mac networks in the cache
```

Then a "WIFICLOUDSYNC" process began, and the first reference to _BrandNewNetwork_ appeared:
```
2025-10-04 14:14:27.440584+0200 0x1d64     Default     0x0                  54     0    wifid: (WiFiCloudSyncEngine) [WIFICLOUDSYNC] __WiFiCloudSyncEngineCheckWaitingForPasswordList (WiFiCloudSyncEngine.m:2651)removed network from waiting for password sync list
2025-10-04 14:14:27.440597+0200 0x1d64     Default     0x0                  54     0    wifid: (WiFiCloudSyncEngine) [WIFICLOUDSYNC] __WiFiCloudSyncEngineCheckWaitingForPasswordList (WiFiCloudSyncEngine.m:2614)3 networks waiting for password sync, currently at 2

2025-10-04 14:14:27.440685+0200 0x1d66     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WiFiNetworkSyncHelperCreateNetworkRef for <BrandNewNetwork>
```

Subsequent log entries showed the network being added to the list of known networks through synchronization.

```
2025-10-04 14:14:27.441055+0200 0x1d66     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WiFiNetworkSyncHelperCreateNetworkRef: network content to be returned to caller: BrandNewNetwork: isHidden=0, isEAP=0, isSAE=0, isWPA=1, isWEP=0, WAPI=0, type=0, enabled=(null), saveData=0, responsiveness=(null) ((null)) isHome=Unknown, isForceFixed=0, transitionDisabledFlags=0, foundNanIe=0, isPH=0, isPublicAirPlayNetwork=0, is6EDisabled=0, hs20=0, Channel=0, isBundleIdentifierPresent=0, allowedBeforeFirstUnlock=-1, PolicyUUID=(null)
2025-10-04 14:14:27.441058+0200 0x1d66     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] __WiFiCloudSyncIsPasswordPresent - for network BrandNewNetwork
2025-10-04 14:14:27.454038+0200 0x1bc2     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WiFiManagerAddNetwork: reason 3, with SSID BrandNewNetwork
2025-10-04 14:14:27.455995+0200 0x1bc2     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] __GetNetworkWithSameSsid: network BrandNewNetwork not found
2025-10-04 14:14:27.455998+0200 0x1bc2     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WiFiNetworkSetDisabledUntilDate: Clearing disable-until property for SSID 'BrandNewNetwork'
2025-10-04 14:14:27.466797+0200 0x1bc2     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] __WiFiDeviceManagerKnownNetworkSuitabilityCheck: Network 'BrandNewNetwork', isFilteringAJCandidates 1, isSSIDTemporarilyDenylisted 0, isBSSIDDenylisted 0, isTDDenylisted 0
2025-10-04 14:14:27.471595+0200 0x1d64     Default     0x0                  54     0    wifid: [WiFiPolicy] {AUTOJOIN*} __WiFiDeviceManagerFilterNetworks: Filtered networks - [...] BrandNewNetwork [...]  ...
2025-10-04 14:14:27.475128+0200 0x1bc1     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WiFiManagerAddNetwork: Added BrandNewNetwork to list of known networks
2025-10-04 14:14:27.475622+0200 0x1acc     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WiFiManagerAddNetwork: <BrandNewNetwork> added due to sync
2025-10-04 14:14:27.477790+0200 0x1d67     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] __WiFiManagerDispatchClientsNetworksChangedEvent: type added network BrandNewNetwork
2025-10-04 14:14:27.477791+0200 0x1d67     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WiFiManagerRemoveNetworkNameFromUserNotificationBlacklist: ssid: BrandNewNetwork
2025-10-04 14:14:27.477795+0200 0x1d67     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:]  WiFiManagerAddPrivateMacNetwork WFMacRandomisation : Checking if network already present : Retrieving private mac cache version of the network <BrandNewNetwork>
2025-10-04 14:14:27.478060+0200 0x1d66     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WiFiManagerAddPrivateMacNetwork WFMacRandomisation : Generated private mac address <<CFData 0x6cd18fca0 [0x1f434aa88]>{length = 6, capacity = 6, bytes = 0xc25657577c60}> for network<BrandNewNetwork> with private MAC mode<3>
2025-10-04 14:14:27.478063+0200 0x1d66     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WiFiNetworkGetProperty: null network
2025-10-04 14:14:27.478065+0200 0x1d66     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WiFiManagerGetPrivateMacNetworksCache:  WFMacRandomisation : Fetched 33 known networks from the cache
2025-10-04 14:14:27.478197+0200 0x1acc     Default     0x0                  54     0    wifid: (CoreWiFi) [com.apple.WiFiManager:] [corewifi] END REQ [GET CURR KNOWN NETWORK] took 0.001321333s (pid=153 proc=networkserviceproxy bundleID=com.apple.networkserviceproxy codesignID=com.apple.networkserviceproxy service=com.apple.private.corewifi-xpc qos=21 intf=(null) uuid=9C5B7 err=0 reply=<redacted>
2025-10-04 14:14:27.478223+0200 0x1acc     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WFMacRandomisation :WiFiManagerAddPrivateMacNetwork:Adding network to private mac cache with reason <11>  existingIndex -1, insertIndex 0
2025-10-04 14:14:27.478225+0200 0x1acc     Default     0x0                  54     0    wifid: (WiFiPolicy) [com.apple.WiFiPolicy:] WiFiManagerSetPrivateMacNetworksCache: Updated 34 private mac networks in the cache
```

Finally, **34** privateMac networks where updated in the cache, one more then 10 minutes before.

## Conclusion
As soon as the iPhone 12 powered on and established a data connection, network synchronization began. The device contained the SSID, password, and timestamp of a Wi-Fi network it had never been near.

This demonstrates that the presence of an SSID in a device’s data does not necessarily mean the phone was ever within range of that network. While the synced data includes the SSID, timestamp, and password, it lacks indicators (such as BSSID or join timestamps) that would confirm an actual connection.
