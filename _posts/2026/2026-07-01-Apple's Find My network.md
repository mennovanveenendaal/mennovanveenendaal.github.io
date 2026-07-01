---
title: "Apple's FindMy network"
layout: post
categories: [iOS, FindMy]
description: Investigating Apple FindMy artefacts in the Apple Unified Log.
image:
  path: /assets/2026/findmy/findmy.png
  alt: FindMy
---

The FindMy network allows supported Apple devices to detect nearby Apple devices and accessories, even when those devices are offline.

According to Apple:

"_If your missing device can’t connect to the internet or has little to no battery life, the Find My app can still help you track it down using the Find My network — over a billion iPhone, iPad, and Mac devices around the world. Nearby devices securely send the location of your missing device to iCloud, then you can see where it is in the Find My app. It’s all anonymous and encrypted to protect everyone’s privacy._" [\[1\]](https://www.apple.com/icloud/find-my/)

A technical description from RealMCU explains the network as:

"_[...]The Apple FindMy network is a crowdsourced network of hundreds of millions of Apple devices that use Bluetooth wireless technology to detect missing devices or items nearby, and report their approximate location back to the owner_" [\[2\]](https://www.realmcu.com/en/Technologies/Protocols/Apple-Find-My-Network)

Most FindMy activity happens entirely in the background. Under normal circumstances, users rarely notice the network operating.

Because of this, I wanted to determine which artifacts are recorded in the Apple Unified Log (AUL) when a device participates in the FindMy network.

## Test setup
For the setup I used an iPhone 12 and an iPhone 16, both running iOS 26.5.

The devices belonged to different Apple accounts and did not share any FindMy relationship. The iPhone 16 was simply placed next to the iPhone 12. No visible interaction occurred between the devices.

## Examining the Apple Unified Log
After placing the devices together, I collected approximately five minutes of Apple Unified Log data.
```shell
# log collect --device iphone12 --start "2026-06-22 17:05:00" --output iphone12_2206.logarchive
```

First I searched for the keyword `findmy` and found activity from `findmybeaconingd` (PID `283`), followed by related activity from `bluetoothd` (PID `97`).

```text
2026-06-22 17:07:14.830218+0200 0x3a4c8    Default     0xc5dc7              283    0    findmybeaconingd: [com.apple.findmy.findmybeaconingd:LocalBeaconingXPCService] <NSXPCConnection: 0x8f0c44320>: beaconingKeys(keyType: 1) ->
[...]
2026-06-22 17:07:14.830306+0200 0x3a4c8    Default     0xc5dc7              283    0    findmybeaconingd: [com.apple.findmy.findmybeaconingd:LocalBeaconingXPCService] <NSXPCConnection: 0x8f0c44320>:   <private> -- 2026-06-22 15:00:00 +0000 to 2026-06-22 15:15:00 +0000
2026-06-22 17:07:14.830324+0200 0x3a4c8    Default     0xc5dc7              283    0    findmybeaconingd: [com.apple.findmy.findmybeaconingd:LocalBeaconingXPCService] <NSXPCConnection: 0x8f0c44320>:   <private> -- 2026-06-22 15:15:00 +0000 to 2026-06-22 15:30:00 +0000
2026-06-22 17:07:14.830340+0200 0x3a4c8    Default     0xc5dc7              283    0    findmybeaconingd: [com.apple.findmy.findmybeaconingd:LocalBeaconingXPCService] <NSXPCConnection: 0x8f0c44320>:   <private> -- 2026-06-22 15:30:00 +0000 to 2026-06-22 15:45:00 +0000
2026-06-22 17:07:14.830357+0200 0x3a4c8    Default     0xc5dc7              283    0    findmybeaconingd: [com.apple.findmy.findmybeaconingd:LocalBeaconingXPCService] <NSXPCConnection: 0x8f0c44320>:   <private> -- 2026-06-22 15:45:00 +0000 to 2026-06-22 16:00:00 +0000
2026-06-22 17:07:14.830372+0200 0x3a4c8    Default     0xc5dc7              283    0    findmybeaconingd: [com.apple.findmy.findmybeaconingd:LocalBeaconingXPCService] <NSXPCConnection: 0x8f0c44320>:   <private> -- 2026-06-22 16:00:00 +0000 to 2026-06-22 16:15:00 +0000
[...]
2026-06-22 17:07:14.830774+0200 0x3acf7    Default     0xcce1e              97     0    bluetoothd: (SPOwner) [com.apple.icloud.SPOwner:beaconManager] Calling beaconingKeyChangedBlock with 5 .nearOwner keys.
```

Several log entries referenced subsystems beginning with `com.apple.findmy.`.

### FindMy Processes and Subsystems
Filtering the Apple Unified Log for `com.apple.findmy.` showed several processes together with their associated subsystems.

- `searchpartyd`
	- `com.apple.findmy.framework.FindMyBluetooth:CBDiscovery`
	- `com.apple.findmy.framework.FindMyBase:XPCActivity`
	- `com.apple.findmy.framework.FindMyBase:transaction`
	- `com.apple.findmy.framework.FindMyBluetooth:centralManager`
- `findmybeaconingd`
	- `com.apple.findmy.framework.FindMyBase:transaction`
	- `com.apple.findmy.framework.FindMyCrypto:crypto`
	- `com.apple.findmy.findmybeaconingd:LocalBeaconingXPCService`
- `runningboardd`
	- `com.apple.runningboard:assertion`

During the five minute capture, `searchpartyd` generated 2420 log entries. `searchpartyd` is the primary daemon responsible for the [offline finding system in iOS](https://arxiv.org/pdf/2103.02282)

![OfflineFinding](/assets/2026/findmy/searchpartyd.png)
_Fig.1 [https://arxiv.org/pdf/2103.02282](https://arxiv.org/pdf/2103.02282)_

The high volume of `searchpartyd` activity makes it one of the most valuable processes to examine when investigating FindMy related events.

## Parsing the Apple Unified Log
Because manually reviewing thousands of log entries is difficult I used Claude to create a small Python script to extract all masked hashes and group them by process and subsystem.

Before running the script, I converted the AUL archive to JSON.

```shell
# log show --archive iphone12_2206.logarchive --info --debug --style json > iphone2206.json
```

```python
# python3 -c "
import json, re
from collections import defaultdict
data = json.load(open('iphone_full.json'))
HASH_RE = re.compile(r\"mask\.hash: '([^']+)'\")
events = []
hash_events = defaultdict(list)
for entry in data:
    subsystem = entry.get('subsystem', '')
    category  = entry.get('category', '')
    message   = entry.get('eventMessage', '')
    timestamp = entry.get('timestamp', '')
    process   = entry.get('processImagePath', '').split('/')[-1]
    hashes    = HASH_RE.findall(message)
    event = {
        'timestamp': timestamp,
        'process':   process,
        'subsystem': subsystem,
        'category':  category,
        'message':   message,
        'hashes':    hashes
    }
    events.append(event)
    for h in hashes:
        hash_events[h].append(event)
# --- Summary ---
from collections import Counter
print('=== SUBSYSTEM BREAKDOWN ===')
for sub, count in Counter(e['subsystem'] for e in events).most_common():
    print(f'  {count:>4}  {sub}')
print()
print('=== SEARCHPARTYD CATEGORY BREAKDOWN ===')
sp_events = [e for e in events if 'searchpartyd' in e['subsystem']]
for cat, count in Counter(e['category'] for e in sp_events).most_common():
    print(f'  {count:>4}  {cat}')
print()
print(f'=== UNIQUE HASHES: {len(hash_events)} ===')
for h, evts in sorted(hash_events.items(), key=lambda x: len(x[1]), reverse=True):
    print(f'  {h}  ({len(evts)} events)')
    print(f'    First: {evts[0][\"timestamp\"]}')
    print(f'    Last:  {evts[-1][\"timestamp\"]}')
    cats = set(e[\"category\"] for e in evts)
    print(f'    Categories: {\", \".join(sorted(cats))}')
"
```

```
=== SUBSYSTEM BREAKDOWN ===
  46208  
  15050  com.apple.network
  13890  com.apple.runningboard
  12944  com.apple.bluetooth
  10648  com.apple.WirelessRadioManager.iRAT
  9397  com.apple.IMAP
  8975  com.apple.xpc
  7415  com.apple.Biome
  5248  com.apple.duetactivityscheduler
  4184  com.apple.networkserviceproxy
  4104  com.apple.UIKit
  4096  com.apple.BiometricKit
  3836  com.apple.containermanager
  3435  com.apple.coreaudio
  3407  com.apple.WiFiPolicy
  3177  com.apple.SpringBoard
  2838  com.apple.locationd.Motion
  2734  com.apple.coreduet
  2538  com.apple.symptomsd
  2297  com.apple.locationd.Core
  2153  com.apple.xpc.activity
  2144  com.apple.mDNSResponder
  2015  com.apple.CFNetwork
  1910  com.apple.powerexperienced
  1891  com.apple.locationd.Position
[...]
   872  com.apple.icloud.searchpartyd
[...]
   141  com.apple.tracking-avoidance
[...]
    26  com.apple.security.keychain.sharing
[...]
    15  com.apple.TrackingAvoidance
[...]
    12  com.apple.findmy.findmybeaconingd
[...]
=== SEARCHPARTYD CATEGORY BREAKDOWN ===
   240  beaconStore
    86  observationStoring
    69  classicPairing
    69  observationStore
    55  centralManager
    46  keyMap
    40  publishScheduler
    25  systemMonitor
    24  baDiscovery
    23  beaconManagerService
    22  multipart
    21  timeBasedKeys
    20  advertisementCache
    19  notifyWhenFound
    17  pencilPairingService
    15  simpleBeaconUpdate
    11  pairingAnalyticsService
     9  beaconPayloadPublishing
     9  CBPeripheralManagement
     9  CBDelegate
     8  productInfoManager
     7  BeaconKeyServiceBuilder
     6  soundManager
     5  command
     5  AnalyticsPublisher
     3  AccessoryConnectionService
     3  separationMonitoring
     2  XPCActivity
     2  accessoryConfigService
     1  ObservationStoreService
     1  BeaconKeyService

=== UNIQUE HASHES: 347 ===
  B2YoOxeCfLmsNyxutXtNBw==  (110 events)
    First: 2026-06-22 17:07:03.992390+0200
    Last:  2026-06-22 17:10:22.362208+0200
    Categories: beaconManagerService, beaconStore, keyMap, notifyWhenFound, observationStore, observationStoring, pairingAnalyticsService, soundManager, timeBasedKeys
[...]
  XG4KAj1wNk4hajtbUCrKHA==  (37 events)
    First: 2026-06-22 17:07:04.015272+0200
    Last:  2026-06-22 17:08:20.893843+0200
    Categories: beaconStore, keyMap, notifyWhenFound, observationStore, observationStoring, pairingAnalyticsService, soundManager, timeBasedKeys
[...]
  cHa9V5QWphclf+NJ3qPB1A==  (10 events)
    First: 2026-06-22 17:07:15.990146+0200
    Last:  2026-06-22 17:10:22.358869+0200
    Categories: observationStoring
[...]
  N3uACnhelU5HKpL228YdPA==  (7 events)
    First: 2026-06-22 17:07:04.016813+0200
    Last:  2026-06-22 17:08:20.892043+0200
    Categories: beaconStore, keyMap, observationStore, productInfoManager, separationMonitoring, soundManager
  33a5nW/Sa+26JzHcwrd4Hw==  (6 events)
    First: 2026-06-22 17:07:03.974262+0200
    Last:  2026-06-22 17:08:20.892721+0200
    Categories: beaconStore, keyMap, observationStore, productInfoManager, soundManager
```
{: .scroll}

This resulted in 974 unique masked hashes. Most appeared in only a single category, such as `dnssd_server`, `Default`, or `Connection`.

### Masked hashes
Apple redacts many sensitive values in the Apple Unified Log. In the FindMy related logs these values appear as `mask.hash` identifiers rather than the original values. In this log for example `<mask.hash: '33a5nW/Sa+26JzHcwrd4Hw=='>, user: <mask.hash: 'Oexh16c5dqEaDtvaTwfX2g=='>`.

Within the captured five minute log window, the same masked identifiers seemed to appear consistently across multiple FindMy related subsystems. This made it possible to correlate related log entries. Because the FindMy protocol rotates advertisement keys approximately [every 15 minutes](https://objectivebythesea.org/v6/talks/OBTS_v6_cFossaceca.pdf),it is unknown whether these masked identifiers remain stable across multiple rotations.

The following log entries appear to show these 15 minute key intervals:
```
2026-06-22 17:07:14.830218+0200 0x3a4c8    Default     0xc5dc7              283    0    findmybeaconingd: [com.apple.findmy.findmybeaconingd:LocalBeaconingXPCService] <NSXPCConnection: 0x8f0c44320>: beaconingKeys(keyType: 1) ->
[...]
2026-06-22 17:07:14.830306+0200 0x3a4c8    Default     0xc5dc7              283    0    findmybeaconingd: [com.apple.findmy.findmybeaconingd:LocalBeaconingXPCService] <NSXPCConnection: 0x8f0c44320>:   <private> -- 2026-06-22 15:00:00 +0000 to 2026-06-22 15:15:00 +0000
2026-06-22 17:07:14.830324+0200 0x3a4c8    Default     0xc5dc7              283    0    findmybeaconingd: [com.apple.findmy.findmybeaconingd:LocalBeaconingXPCService] <NSXPCConnection: 0x8f0c44320>:   <private> -- 2026-06-22 15:15:00 +0000 to 2026-06-22 15:30:00 +0000
2026-06-22 17:07:14.830340+0200 0x3a4c8    Default     0xc5dc7              283    0    findmybeaconingd: [com.apple.findmy.findmybeaconingd:LocalBeaconingXPCService] <NSXPCConnection: 0x8f0c44320>:   <private> -- 2026-06-22 15:30:00 +0000 to 2026-06-22 15:45:00 +0000
2026-06-22 17:07:14.830357+0200 0x3a4c8    Default     0xc5dc7              283    0    findmybeaconingd: [com.apple.findmy.findmybeaconingd:LocalBeaconingXPCService] <NSXPCConnection: 0x8f0c44320>:   <private> -- 2026-06-22 15:45:00 +0000 to 2026-06-22 16:00:00 +0000
2026-06-22 17:07:14.830372+0200 0x3a4c8    Default     0xc5dc7              283    0    findmybeaconingd: [com.apple.findmy.findmybeaconingd:LocalBeaconingXPCService] <NSXPCConnection: 0x8f0c44320>:   <private> -- 2026-06-22 16:00:00 +0000 to 2026-06-22 16:15:00 +0000
[...]
2026-06-22 17:07:14.830557+0200 0x3a4c8    Info        0xc5dc7              283    0    findmybeaconingd: (FindMyBase) [com.apple.findmy.framework.FindMyBase:transaction] Closed [TXN:beaconingKeys(keyType:withCompletion:).C7202D97-9171-4430-94F3-DBDA38A1CF83]
```

## Analyzing masked hashes
### User hash
To reduce the amount of data, I filtered the log to only include entries generated by `findmybeaconingd`, `searchpartyd`, and `bluetoothd`. This filtered dataset contained one unique user hash: `Oexh16c5dqEaDtvaTwfX2g==`.
```
2026-06-22 17:07:03.974262+0200 0x3ad63    Default     0xcdc00              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStore] Decimated 3 redundant battery records for beacon: <mask.hash: '33a5nW/Sa+26JzHcwrd4Hw=='>, user: <mask.hash: 'Oexh16c5dqEaDtvaTwfX2g=='>, keeping 2 records.
2026-06-22 17:07:03.992390+0200 0x3ad63    Default     0xcdc00              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStore] Decimated 22 redundant battery records for beacon: <mask.hash: 'B2YoOxeCfLmsNyxutXtNBw=='>, user: <mask.hash: 'Oexh16c5dqEaDtvaTwfX2g=='>, keeping 37 records.
2026-06-22 17:07:04.015272+0200 0x3ad63    Default     0xcdc00              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStore] Decimated 16 redundant battery records for beacon: <mask.hash: 'XG4KAj1wNk4hajtbUCrKHA=='>, user: <mask.hash: 'Oexh16c5dqEaDtvaTwfX2g=='>, keeping 44 records.
2026-06-22 17:07:04.016813+0200 0x3ad63    Default     0xcdc00              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStore] Decimated 1 redundant battery records for beacon: <mask.hash: 'N3uACnhelU5HKpL228YdPA=='>, user: <mask.hash: 'Oexh16c5dqEaDtvaTwfX2g=='>, keeping 2 records.
```

### Beacon analysis
Filtering on these beacon hashes in the results of the Python script:

| Hash                       | Events | Categories                                                                                                                                                               |
| -------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `B2YoOxeCfLmsNyxutXtNBw==` | 110    | `beaconManagerService`, `beaconStore`, `keyMap`, `notifyWhenFound`, `observationStore`, `observationStoring`, `pairingAnalyticsService`, `soundManager`, `timeBasedKeys` |
| `XG4KAj1wNk4hajtbUCrKHA==` | 37     | `beaconStore`, `keyMap`, `notifyWhenFound`, `observationStore`, `observationStoring`, `pairingAnalyticsService`, `soundManager`, `timeBasedKeys`                         |
| `N3uACnhelU5HKpL228YdPA==` | 7      | `beaconStore`, `keyMap`, `observationStore`, `productInfoManager`, `separationMonitoring`, `soundManager`                                                                |
| `33a5nW/Sa+26JzHcwrd4Hw==` | 6      | `beaconStore`, `keyMap`, `observationStore`, `productInfoManager`, `soundManager`                                                                                        | 

#### Beacon `B2YoOxeCfLmsNyxutXtNBw==`
The log entries suggest that this beacon is associated with advertisement `cHa9V5QWphclf+NJ3qPB1A==`.
```
2026-06-22 17:07:15.990146+0200 0x3ad64    Info        0xcf17e              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStoring] "Reconciled advertisement <mask.hash: 'cHa9V5QWphclf+NJ3qPB1A=='>, multiPart: 100, hint: none, Type18: {type: hele, battery: high, maintained: true multipart: 100},
index: .primary/(50830), beacon: <mask.hash: 'B2YoOxeCfLmsNyxutXtNBw=='>,
hasLocation: true.
2026-06-22 17:07:15.990183+0200 0x3a97d    Default     0x0                  370    0    ConnectMobile: (libusrtcp.dylib) [com.apple.network:tcp] tcp_input [C285.1.1.1:3] flags=[F.] seq=802961322, ack=3958965034, win=16 state=FIN_WAIT_1 rcv_nxt=802961322, snd_una=3958965033
2026-06-22 17:07:15.990236+0200 0x3ac90    Activity    0xcf17f              228    0    searchpartyd: (D194B574-D1A5-3B7F-8FB6-FA9EAAC80112) 
2026-06-22 17:07:15.990242+0200 0x3ac90    Info        0xcf17f              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Opened [TXN:searchpartyd.beaconstore.updateType18Status.A0FFC614-2063-46AB-A29F-7C31A53F2350]
2026-06-22 17:07:15.990323+0200 0x3ac90    Info        0xcf17f              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] Latest type18 for <mask.hash: 'B2YoOxeCfLmsNyxutXtNBw=='>: {type: hele, battery: high, maintained: true multipart: 100},scanDate: 2026-06-22 15:07:16 +0000,sequence: .primary,index: 50830, appActive: false.
2026-06-22 17:07:15.990339+0200 0x3ac90    Default     0xcf17f              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] wildModeAssociationRecord(for uuid: <private>)
2026-06-22 17:07:15.990678+0200 0x3b0b8    Info        0xcf17d              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStoring] "Reconciled advertisement <mask.hash: 'cHa9V5QWphclf+NJ3qPB1A=='>, multiPart: 100, hint: none, Type18: {type: hele, battery: high, maintained: true multipart: 100},
index: .primary/(50830), beacon: <mask.hash: 'B2YoOxeCfLmsNyxutXtNBw=='>,
hasLocation: false.
2026-06-22 17:07:15.990742+0200 0x3b0b8    Activity    0xcf2e0              228    0    searchpartyd: (D194B574-D1A5-3B7F-8FB6-FA9EAAC80112) 
2026-06-22 17:07:15.990745+0200 0x3b0b8    Info        0xcf2e0              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Opened [TXN:searchpartyd.beaconstore.updateType18Status.4EADE4F5-7D2D-44F7-A0B3-8A9604DD2CC0]
2026-06-22 17:07:15.990798+0200 0x3ac90    Info        0xcf17f              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] beaconRecord(for uuid: <private>)
2026-06-22 17:07:15.991264+0200 0x3ac90    Activity    0xcf2e1              228    0    searchpartyd: (D194B574-D1A5-3B7F-8FB6-FA9EAAC80112) 
2026-06-22 17:07:15.991268+0200 0x3ac90    Info        0xcf2e1              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Opened [TXN:SimpleBeaconStatusUpdate.C453663C-946E-40F7-8A09-38256157B443]
2026-06-22 17:07:15.991282+0200 0x3ac90    Info        0xcf2e1              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Closed [TXN:searchpartyd.beaconstore.updateType18Status.A0FFC614-2063-46AB-A29F-7C31A53F2350]
2026-06-22 17:07:15.991391+0200 0x3ac90    Info        0xcf2e0              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Closed [TXN:searchpartyd.beaconstore.updateType18Status.4EADE4F5-7D2D-44F7-A0B3-8A9604DD2CC0]
2026-06-22 17:07:15.991450+0200 0x3ae0f    Default     0x0                  86     0    passd: (CoreSpotlight) [com.apple.corespotlight:query] [qid=470969011119587334] Results type: SPQueryResults
2026-06-22 17:07:15.991534+0200 0x3ac90    Info        0xcf2e1              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Closed [TXN:SimpleBeaconStatusUpdate.C453663C-946E-40F7-8A09-38256157B443]
2026-06-22 17:07:15.991744+0200 0x3ae0f    Default     0x0                  86     0    passd: (CoreSpotlight) [com.apple.corespotlight:query] [qid=470969011119587334] Found: 29 items
2026-06-22 17:07:15.991762+0200 0x3b0b8    Info        0xcf17d              228    0    searchpartyd: [com.apple.icloud.searchpartyd:CBPeripheralManagement] ObservedAdvertisement for <private>/<private>/<private>already exists (advId=1424549).
2026-06-22 17:07:15.992414+0200 0x3ad64    Info        0xcf17e              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStore] Skipping to update key sync metadata for <mask.hash: 'B2YoOxeCfLmsNyxutXtNBw=='>, type: .nearOwner, .primary(50830).
2026-06-22 17:07:15.992811+0200 0x3ad63    Info        0xcf17d              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStore] Skipping to update key sync metadata for <mask.hash: 'B2YoOxeCfLmsNyxutXtNBw=='>, type: .nearOwner, .primary(50830).
2026-06-22 17:07:15.992981+0200 0x3ad64    Info        0xcf17e              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStoring] Not a managed periphereral: <mask.hash: 'B2YoOxeCfLmsNyxutXtNBw=='>
2026-06-22 17:07:15.993247+0200 0x3ad64    Info        0xcf17e              228    0    searchpartyd: [com.apple.icloud.searchpartyd:notifyWhenFound] No NWF record for beacon <mask.hash: 'B2YoOxeCfLmsNyxutXtNBw=='>. No need to publish
2026-06-22 17:07:15.993546+0200 0x3ad63    Info        0xcf17d              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStoring] Not a managed periphereral: <mask.hash: 'B2YoOxeCfLmsNyxutXtNBw=='>
2026-06-22 17:07:15.993727+0200 0x3b0b8    Info        0xcf17d              228    0    searchpartyd: [com.apple.icloud.searchpartyd:notifyWhenFound] No NWF record for beacon <mask.hash: 'B2YoOxeCfLmsNyxutXtNBw=='>. No need to publish
2026-06-22 17:07:15.994065+0200 0x3ad63    Info        0xcf17e              228    0    searchpartyd: [com.apple.icloud.searchpartyd:publishScheduler] FindMyNetworkPublishActivityService maintaining existing criteria.
2026-06-22 17:07:15.994169+0200 0x3ad63    Info        0xcf17e              228    0    searchpartyd: (FindMyBase) [com.apple.findmy.framework.FindMyBase:transaction] Closed [TXN:SaveObservedAdvertisment.0B68D312-33EA-4892-92BA-3524D1CDE8B9]
2026-06-22 17:07:15.994530+0200 0x3ad64    Info        0xcf17d              228    0    searchpartyd: [com.apple.icloud.searchpartyd:publishScheduler] FindMyNetworkPublishActivityService maintaining existing criteria.
2026-06-22 17:07:15.994590+0200 0x3ad64    Info        0xcf17d              228    0    searchpartyd: (FindMyBase) [com.apple.findmy.framework.FindMyBase:transaction] Closed [TXN:SaveObservedAdvertisment.DC664E06-418A-4C01-8EB6-7175B43E1363]
```
{: .scroll}

#### Beacon `XG4KAj1wNk4hajtbUCrKHA==`
The log entries suggest that this beacon is associated with advertisement `TYdBykst109NRV1MEIbwAw==`.

```
2026-06-22 17:07:14.218954+0200 0x3ac90    Default     0xcea18              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] wildModeAssociationRecords(advertisement: <private>)
2026-06-22 17:07:14.219373+0200 0x3ad64    Default     0xcea17              228    0    searchpartyd: (FindMyBase) [com.apple.icloud.searchpartyd:observationStoring] TIME: ObservationStoring.Reconcile: <private>
2026-06-22 17:07:14.221602+0200 0x3ad64    Info        0xcea17              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStoring] "Reconciled advertisement <mask.hash: 'TYdBykst109NRV1MEIbwAw=='>, multiPart: 010, hint: 93, Type18: {type: hele, battery: high, maintained: false multipart: 010},
index: .secondary/(428), beacon: <mask.hash: 'XG4KAj1wNk4hajtbUCrKHA=='>,
hasLocation: false.
2026-06-22 17:07:14.221613+0200 0x3ad63    Default     0xcea18              228    0    searchpartyd: (FindMyBase) [com.apple.icloud.searchpartyd:observationStoring] TIME: ObservationStoring.Reconcile: <private>
2026-06-22 17:07:14.221735+0200 0x3ac90    Activity    0xcea19              228    0    searchpartyd: (D194B574-D1A5-3B7F-8FB6-FA9EAAC80112) 
2026-06-22 17:07:14.221745+0200 0x3ac90    Info        0xcea19              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Opened [TXN:searchpartyd.beaconstore.updateType18Status.394630AC-770C-4121-A719-A2C57A160578]
2026-06-22 17:07:14.221883+0200 0x3ac90    Info        0xcea19              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] Latest type18 for <mask.hash: 'XG4KAj1wNk4hajtbUCrKHA=='>: {type: hele, battery: high, maintained: false multipart: 010},scanDate: 2026-06-22 15:07:14 +0000,sequence: .secondary,index: 428, appActive: false.
2026-06-22 17:07:14.221913+0200 0x3ac90    Default     0xcea19              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] wildModeAssociationRecord(for uuid: <private>)
2026-06-22 17:07:14.222611+0200 0x3ac90    Info        0xcea19              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] beaconRecord(for uuid: <private>)
2026-06-22 17:07:14.223261+0200 0x3ad63    Info        0xcea18              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStoring] "Reconciled advertisement <mask.hash: 'TYdBykst109NRV1MEIbwAw=='>, multiPart: 010, hint: 93, Type18: {type: hele, battery: high, maintained: false multipart: 010},
index: .secondary/(428), beacon: <mask.hash: 'XG4KAj1wNk4hajtbUCrKHA=='>,
hasLocation: true.
2026-06-22 17:07:14.223322+0200 0x3ac90    Activity    0xcea1a              228    0    searchpartyd: (D194B574-D1A5-3B7F-8FB6-FA9EAAC80112) 
2026-06-22 17:07:14.223329+0200 0x3ac90    Info        0xcea1a              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Opened [TXN:SimpleBeaconStatusUpdate.0BF4207C-C0C0-4BEB-BF3D-CDFCEB22206B]
2026-06-22 17:07:14.223360+0200 0x3ac90    Info        0xcea1a              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Closed [TXN:searchpartyd.beaconstore.updateType18Status.394630AC-770C-4121-A719-A2C57A160578]
2026-06-22 17:07:14.223365+0200 0x3ad63    Activity    0xcea1b              228    0    searchpartyd: (D194B574-D1A5-3B7F-8FB6-FA9EAAC80112) 
2026-06-22 17:07:14.223371+0200 0x3ad63    Info        0xcea1b              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Opened [TXN:searchpartyd.beaconstore.updateType18Status.3C4F862D-EC3A-45E5-8E39-0421679AA57B]
2026-06-22 17:07:14.223510+0200 0x3ac90    Info        0xcea1b              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Closed [TXN:searchpartyd.beaconstore.updateType18Status.3C4F862D-EC3A-45E5-8E39-0421679AA57B]
2026-06-22 17:07:14.223688+0200 0x3ad63    Info        0xcea1a              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Closed [TXN:SimpleBeaconStatusUpdate.0BF4207C-C0C0-4BEB-BF3D-CDFCEB22206B]
2026-06-22 17:07:14.223740+0200 0x3ad63    Info        0xcea17              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] ownedBeaconRecord(for uuid: <private>)
2026-06-22 17:07:14.224170+0200 0x3ac90    Info        0xcea18              228    0    searchpartyd: [com.apple.icloud.searchpartyd:CBPeripheralManagement] ObservedAdvertisement for <private>/<private>/<private>already exists (advId=1424548).
```
{: .scroll}

#### Beacon `N3uACnhelU5HKpL228YdPA==`
The logs indicate that this beacon represents the local device, preventing separation monitoring.

```
2026-06-22 17:07:04.016813+0200 0x3ad63    Default     0xcdc00              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStore] Decimated 1 redundant battery records for beacon: <mask.hash: 'N3uACnhelU5HKpL228YdPA=='>, user: <mask.hash: 'Oexh16c5dqEaDtvaTwfX2g=='>, keeping 2 records.
2026-06-22 17:07:14.706789+0200 0x3ad67    Default     0xcf174              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] Beacon <mask.hash: 'N3uACnhelU5HKpL228YdPA=='> is not connected. Last seen: 0001-01-01 00:00:00 +0000.
2026-06-22 17:07:14.708059+0200 0x3ad67    Default     0xcf174              228    0    searchpartyd: [com.apple.icloud.searchpartyd:separationMonitoring] Can't monitor beacon: <mask.hash: 'N3uACnhelU5HKpL228YdPA=='> due to: this device.
```
	  
#### Beacon  `33a5nW/Sa+26JzHcwrd4Hw==` 
The logs indicate that this beacon was last observed before the log collection started.
```
2026-06-22 17:07:03.974262+0200 0x3ad63    Default     0xcdc00              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStore] Decimated 3 redundant battery records for beacon: <mask.hash: '33a5nW/Sa+26JzHcwrd4Hw=='>, user: <mask.hash: 'Oexh16c5dqEaDtvaTwfX2g=='>, keeping 2 records.
2026-06-22 17:07:14.739651+0200 0x3ac90    Default     0xcf174              228    0    searchpartyd: [com.apple.icloud.searchpartyd:productInfoManager] No productData for <mask.hash: '33a5nW/Sa+26JzHcwrd4Hw=='>
2026-06-22 17:07:14.740287+0200 0x3ac90    Default     0xcf174              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] Beacon <mask.hash: '33a5nW/Sa+26JzHcwrd4Hw=='> is not connected. Last seen: 2026-06-22 14:59:17 +0000.
2026-06-22 17:07:14.744770+0200 0x3ad67    Default     0xcf174              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] Beacon <mask.hash: '33a5nW/Sa+26JzHcwrd4Hw=='> is not connected. Last seen: 2026-06-22 14:59:17 +0000.
2026-06-22 17:07:14.749165+0200 0x3ac90    Info        0xcf174              228    0    searchpartyd: [com.apple.icloud.searchpartyd:soundManager] TaskInfo for Beacon: <mask.hash: '33a5nW/Sa+26JzHcwrd4Hw=='> has state: .idle.
2026-06-22 17:08:20.892721+0200 0x3b37d    Info        0xcfb0a              228    0    searchpartyd: [com.apple.icloud.searchpartyd:keyMap] KeyMap reconciler for owned beacon <mask.hash: '33a5nW/Sa+26JzHcwrd4Hw=='>.
```

## The `notifyWhenFound` Subsystem
Two of the hashes were found in the `notifyWhenFound` subsystem. 

| Hash                       | Events | Categories                                                                                                                                                               |
| -------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `B2YoOxeCfLmsNyxutXtNBw==` | 110    | `beaconManagerService`, `beaconStore`, `keyMap`, `notifyWhenFound`, `observationStore`, `observationStoring`, `pairingAnalyticsService`, `soundManager`, `timeBasedKeys` |
| `XG4KAj1wNk4hajtbUCrKHA==` | 37     | `beaconStore`, `keyMap`, `notifyWhenFound`, `observationStore`, `observationStoring`, `pairingAnalyticsService`, `soundManager`, `timeBasedKeys`                         |

### Beacons `B2YoOxeCfLmsNyxutXtNBw==` and `XG4KAj1wNk4hajtbUCrKHA==`
Both where found in the `notifyWhenFound` subsystem. When searching for these hashes I found the logging sequence to be about the same for both.

```
2026-06-22 17:07:16.617866+0200 0x3ad63    Info        0xcf2e4              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Opened [TXN:searchpartyd.beaconstore.updateType18Status.2F0486BA-483A-4C04-8B7B-A6E08D7AC5E3]
2026-06-22 17:07:16.618010+0200 0x3b0b8    Info        0xcf2e4              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] Latest type18 for <mask.hash: 'XG4KAj1wNk4hajtbUCrKHA=='>: {type: hele, battery: high, maintained: false multipart: 010},scanDate: 2026-06-22 15:07:16 +0000,sequence: .secondary,index: 428, appActive: false.
2026-06-22 17:07:16.618023+0200 0x3b0b8    Default     0xcf2e4              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] wildModeAssociationRecord(for uuid: <private>)
2026-06-22 17:07:16.618657+0200 0x3b0b8    Info        0xcf2e4              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] beaconRecord(for uuid: <private>)
2026-06-22 17:07:16.619204+0200 0x3ad64    Info        0xcf2e3              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStoring] "Reconciled advertisement <mask.hash: 'TYdBykst109NRV1MEIbwAw=='>, multiPart: 010, hint: 93, Type18: {type: hele, battery: high, maintained: false multipart: 010},
2026-06-22 17:07:16.619320+0200 0x3ad67    Activity    0xcf2e5              228    0    searchpartyd: (D194B574-D1A5-3B7F-8FB6-FA9EAAC80112) 
2026-06-22 17:07:16.619327+0200 0x3ad67    Info        0xcf2e5              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Opened [TXN:searchpartyd.beaconstore.updateType18Status.25E98896-D036-4276-88E9-F40F4DF4501D]
2026-06-22 17:07:16.619386+0200 0x3b0b8    Activity    0xcf2e6              228    0    searchpartyd: (D194B574-D1A5-3B7F-8FB6-FA9EAAC80112) 
2026-06-22 17:07:16.619392+0200 0x3b0b8    Info        0xcf2e6              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Opened [TXN:SimpleBeaconStatusUpdate.3DE99A11-D466-4FA4-9421-3040040962D5]
2026-06-22 17:07:16.619414+0200 0x3b0b8    Info        0xcf2e6              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Closed [TXN:searchpartyd.beaconstore.updateType18Status.2F0486BA-483A-4C04-8B7B-A6E08D7AC5E3]
2026-06-22 17:07:16.619561+0200 0x3b0b8    Info        0xcf2e5              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Closed [TXN:searchpartyd.beaconstore.updateType18Status.25E98896-D036-4276-88E9-F40F4DF4501D]
2026-06-22 17:07:16.619658+0200 0x3b0b8    Info        0xcf2e6              228    0    searchpartyd: (SPShared) [com.apple.icloud.spshared:transaction] Closed [TXN:SimpleBeaconStatusUpdate.3DE99A11-D466-4FA4-9421-3040040962D5]
2026-06-22 17:07:16.619803+0200 0x3ad63    Info        0xcf2e2              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] ownedBeaconRecord(for uuid: <private>)
2026-06-22 17:07:16.620203+0200 0x3b0b8    Info        0xcf2e3              228    0    searchpartyd: [com.apple.icloud.searchpartyd:CBPeripheralManagement] ObservedAdvertisement for <private>/<private>/<private>already exists (advId=1424550).
2026-06-22 17:07:16.621116+0200 0x3b0b8    Info        0xcf2e3              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] ownedBeaconRecord(for uuid: <private>)
2026-06-22 17:07:16.634313+0200 0x3b0b8    Default     0xcf2e2              228    0    searchpartyd: [com.apple.icloud.searchpartyd:keyMap] BeaconKeyManager: keys(beacon:startBucket:endBucket:sequence:forceGenerate:useDaemonDirectory:). Keys for Beacon <private>, sequence .primary, index 40992 - 41183
2026-06-22 17:07:16.634817+0200 0x3ad63    Default     0xcf2e2              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] Found 3 matching hint advertisements for beacon <mask.hash: 'XG4KAj1wNk4hajtbUCrKHA=='>.
2026-06-22 17:07:16.635148+0200 0x3b0b8    Info        0xcf2e2              228    0    searchpartyd: [com.apple.icloud.searchpartyd:timeBasedKeys] Bucket [.primary] calculated to ‣1 baseTime: 2026-06-22 15:07:16 +0000 date: 2026-06-22 15:07:16 +0000  beacon: <mask.hash: 'XG4KAj1wNk4hajtbUCrKHA=='>.
2026-06-22 17:07:16.635577+0200 0x3b0b8    Info        0xcf2e2              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStore] Skipping to update key sync metadata for <mask.hash: 'XG4KAj1wNk4hajtbUCrKHA=='>, type: .hintBased, .primary(41007).
2026-06-22 17:07:16.636154+0200 0x3b0b8    Info        0xcf2e2              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStoring] Not a managed periphereral: <mask.hash: 'XG4KAj1wNk4hajtbUCrKHA=='>
2026-06-22 17:07:16.642936+0200 0x3ac90    Default     0xcf2e3              228    0    searchpartyd: [com.apple.icloud.searchpartyd:keyMap] BeaconKeyManager: keys(beacon:startBucket:endBucket:sequence:forceGenerate:useDaemonDirectory:). Keys for Beacon <private>, sequence .primary, index 40992 - 41183
2026-06-22 17:07:16.643428+0200 0x3ad63    Default     0xcf2e3              228    0    searchpartyd: [com.apple.icloud.searchpartyd:beaconStore] Found 3 matching hint advertisements for beacon <mask.hash: 'XG4KAj1wNk4hajtbUCrKHA=='>.
2026-06-22 17:07:16.643782+0200 0x3ac90    Info        0xcf2e3              228    0    searchpartyd: [com.apple.icloud.searchpartyd:timeBasedKeys] Bucket [.primary] calculated to ‣1 baseTime: 2026-06-22 15:07:16 +0000 date: 2026-06-22 15:07:16 +0000  beacon: <mask.hash: 'XG4KAj1wNk4hajtbUCrKHA=='>.
2026-06-22 17:07:16.643866+0200 0x3ad63    Info        0xcf2e2              228    0    searchpartyd: [com.apple.icloud.searchpartyd:notifyWhenFound] No NWF record for beacon <mask.hash: 'XG4KAj1wNk4hajtbUCrKHA=='>. No need to publish
2026-06-22 17:07:16.644212+0200 0x3ac90    Info        0xcf2e3              228    0    searchpartyd: [com.apple.icloud.searchpartyd:observationStore] Skipping to update key sync metadata for <mask.hash: 'XG4KAj1wNk4hajtbUCrKHA=='>, type: .hintBased, .primary(41007).
2026-06-22 17:07:16.644509+0200 0x3ad63    Info        0xcf2e2              228    0    searchpartyd: [com.apple.icloud.searchpartyd:publishScheduler] FindMyNetworkPublishActivityService maintaining existing criteria.
2026-06-22 17:07:16.644595+0200 0x3ad63    Info        0xcf2e2              228    0    searchpartyd: (FindMyBase) [com.apple.findmy.framework.FindMyBase:transaction] Closed [TXN:SaveObservedAdvertisment.142CE6BF-D821-4D42-A88F-6EA0F3310C8E]
```
{: .scroll}

The final log entries indicate that no `NWF record` exists for this beacon. NWF appears to be an abbreviation for _Notify When Found_. If no NWF record exists, the logs indicate there is no need to publish an event.

## Findings
Although the Apple Unified Log does not expose the identity of nearby devices, it does reveal a considerable amount of FindMy processing. Because Apple masks many values inside the Apple Unified Log, it is not possible to definitively associate a beacon with a specific physical device using the log alone. 

Based on this log it is likely that beacon `B2YoOxeCfLmsNyxutXtNBw==` or `XG4KAj1wNk4hajtbUCrKHA==` is related to the iPhone 16. In this short timespan this device was seen and also processed in the `notifyWhenFound` subsystem.