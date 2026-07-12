---
title: "AirPods and the Apple Unified Log. Part 2. Opening the Charging Case"
layout: post
categories: [iOS, AirPods]
description: Investigating the Apple Unified Logging to find artefacts of connected Airpods. Part 2.
image:
  path: /assets/2026/airpods/airpods_2.png
  alt: FindMy
---

This post is the second in a serie that examines which AirPods related artefacts are recorded in the Apple Unified Log. This post focuses on opening of the Charging Case.

## Opening the Charging Case
Prior to opening the Charging Case, no AirPods related log entries were observed within the selected time window. When the Charging Case was opened the logging of the `com.apple.AudioAccessory` subsystem stated that the AirPods were detected as a paired device.

```
2026-06-29 16:19:01.591564+0200 0x3cba0    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:SR3PRoutingDaemon] SR3P Pairing found device CBDevice F4CF7039-6E99-F1D6-DF03-77A730FB317B, BDA <private>, Nm <private> , Md AirPodsPro1,1, PID 0x200E (AirPodsPro1,1), PrNm AirPods Pro, VID 0x004C, VS 1, DsFl 0x40808000 < WxStatus Pairing BLEAdvertisementData >, DvF 0x5001D80403F < AACP MagicPaired ShareAudio SoftwareVolume Tipi PSE ClassicPaired SpatialAudio AdvancedAppleAudio ANC Transparency SpatialAudioAllowed SmartRouting Connectable >, DvT Headphones, RSSI -37, SupS 0x180019 < HFP AVRCP A2DP AACP GATT >, AStS Idle, Freq 2.4, Ch 38, ClkH L NoiseManagement R NoiseManagement, ECC 2, MCCp 1, MCC 1, CVer'1.4.1', DbTp C Basic, GAPA 0x1 < Real >, FV '6F21', LsnM Normal, LsMC 0x6 < ANC Transparency >, BTv 5.0, MicM Auto, Plcm M Enabled, Prim Unknown, modU A2084, SN 'H6RFJ7190C6L', SN Left'H6RFK5050C6K', SN Right 'H6VFKFXY0C6J', srMd Enabled, spAM ContentDriven, AdTsMC <30606500471>, AMfD <4c 00 07 19 01 0e 20 04 f9 99 31 00 00 00 df ff 61 ca bf d7 70 ff 0b 00 10 b1 13 a9 e3>, ppPI 0x200E (AirPodsPro1,1), ppST 0x01 (WxSta<…>
```

The log reports the active noise control mode as Transparency (`ANC Transparency`).

```
2026-06-29 16:19:01.591636+0200 0x3cb9d    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AANearbyDeviceManagerDaemon] Creating os transaction for nearby devices.
```

## Device identifiers
```
2026-06-29 16:19:01.592639+0200 0x3cb9d    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AANearbyDeviceManagerDaemon] AANearbyDevice found from CBDiscovery: AANearbyDevice identifier: F4CF7039-6E99-F1D6-DF03-77A730FB317B, Bt addr 'B8:81:FA:11:FF:AB', Md 'AirPodsPro1,1', Nm 'AirPods Pro', paired
```

## Advertisement payload
```
2026-06-29 16:19:01.592846+0200 0x3cb9d    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AANearbyDeviceManagerDaemon] AANearbyDevice updated from advertisement payload: AAProximityPairingStatusPayloadB188B288, PT: 7, PID 8206, ST: 1, Sup Wir Split: yes, Audio St: Idle, Src count: 0, DEOC on: no, OBC on: no, SR Conn: no, My Batt: +95, Oth Batt: invalid, C Batt: -97, lst conn hst: cabfd7, lst bud in C w/curr bud: 70ff0b, SR score 1: Unknown, SR score 2: Unknown, Idle time: <5s, time out of case: <5s, lst conn host iCloud signed in: yes, UTP: no, This: Right - Primary - In Case, Other: Unknown, Out of Box: no, L Batt: invalid, R Batt: +95%, lid open count: 1, lid closed: no, case ver: B435, case led col: Green, case led status: Solid, color: 0, Adv Data: <07 19 01 0e 20 04 f9 99 31 00 00 00 df ff 61 ca bf d7 70 ff 0b 00 10 b1 13 a9 e3>
```

This logline logged the "advertisement payload":
- `This: Right - Primary - In Case`
- `Other: Unknown`
- `Out of Box: no`
- `R Batt: +95%`
- `lid open count: 1`
- `lid closed: no`

At this stage, the Charging Case was open. The advertisement payload indicates that the primary (right) AirPod was still in the case, while the state of the secondary (left) AirPod remained unknown.

## Charging Case detected
The following entries indicate that the Charging Case has been detected.

```
2026-06-29 16:19:01.600998+0200 0x3cb0f    Default     0xcc00f              97     0    bluetoothd: (WPDaemon) [com.apple.bluetooth:WirelessProximity] Scan options changed: 1

2026-06-29 16:19:01.601261+0200 0x3cb0f    Default     0xcc00f              97     0    bluetoothd: [com.apple.bluetooth:Server.XPC] Received XPC message "CBMsgIdScan" from session "com.apple.bluetoothd-central-97-9"

2026-06-29 16:19:01.601441+0200 0xeda8     Default     0x0                  1539   0    audioaccessoryd: (HealthKit) [com.apple.HealthKit:infrastructure] [HKBluetoothDeviceDataSource] Device found: F4CF7039-6E99-F1D6-DF03-77A730FB317B
```

## Battery publication
The following log records the battery information that has been updated.

```
2026-06-29 16:19:01.602102+0200 0x3cb9d    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AADevicePowerSources] Power source publish update for battery: 'Case' with details:
{
    "Max Capacity" : 100,
    "Low Warn Level" : 25,
    "Is Present" : true,
    "Is Charged" : 0,
    "Name" : "AirPods Pro Case",
    "Type" : "Accessory Source",
    "Accessory Identifier" : "F4CF7039-6E99-F1D6-DF03-77A730FB317B",
    "Accessory Category" : "Audio Battery Case",
    "Part Identifier" : "Case",
    "Is Charging" : false,
    "Time to Full Charge" : 0,
    "Transport Type" : "Bluetooth LE",
    "LEDs" : 
    [
        {
            "State" : "Solid",
            "Color" : "Green",
        }
    ],
    "Current Capacity" : 97,
    "Vendor ID Source" : 1,
    "Optimized Battery Charging Engaged" : false,
    "Vendor ID" : 76,
    "Dynamic End of Charging Engaged" : false,
    "Group Identifier" : "F4CF7039-6E99-F1D6-DF03-77A730FB317B",
    "Product ID" : 8206,
    "Power Source State" : "Battery Power",
}
```

## Charging Case and AirPods detected
Shortly afterwards, battery information is published for both individual AirPods as well as the Charging Case.
 `Batteries: C, LR, `.

```
2026-06-29 16:19:01.734285+0200 0x3cba0    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AADevicePowerSources] AADevicePowerSource, Batteries: C, LR, 
Details:
Case: {
    "Max Capacity" : 100,
    "Low Warn Level" : 25,
    "Is Present" : true,
    "Is Charged" : 0,
    "Name" : "AirPods Pro Case",
    "Type" : "Accessory Source",
    "Accessory Identifier" : "F4CF7039-6E99-F1D6-DF03-77A730FB317B",
    "Accessory Category" : "Audio Battery Case",
    "Part Identifier" : "Case",
    "Is Charging" : false,
    "Time to Full Charge" : 0,
    "Transport Type" : "Bluetooth LE",
    "LEDs" : 
    [
        {
            "State" : "Solid",
            "Color" : "Off",
        }
    ],
    "Current Capacity" : 97,
    "Vendor ID Source" : 1,
    "Optimized Battery Charging Engaged" : false,
    "Vendor ID" : 76,
    "Dynamic End of Charging Engaged" : false,
    "Group Identifier" : "F4CF7039-6E99-F1D6-DF03-77A730FB317B",
    "Product ID" : 8206,
    "Power Source State" : "Battery Power",
}

LeftRight: {
    "Max Capacity" : 100,
    "Combined Parts" : 
    [
        {
            "Max Capacity" : 100,
            <…>
```

## Overview of the articles
- [Part 1. Introduction and methodology](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-1.-Introduction-and-methodology/)
- Part 2. Opening the Charging Case
- [Part 3. Detecting AirPods removed and inserted into the ear](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-3.-Detecting-AirPods-removed-and-inserted-into-the-ear/)
- [Part 4. Closing the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-4.-Closing-the-Charging-Case/)
- [Part 5. Media play event](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-5.-Media-play-event/)
- [Part 6. Media pause events](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-6.-Media-pausing-event/)
- [Part 7. Detecting AirPods removed from the ears](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-7.-Detecting-AirPods-removed-from-the-ears/)
- [Part 8. Detecting AirPods returned to the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-8.-Detecting-AirPods-returned-to-the-Charging-Case/)
- [Part 9. Closing the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-9.-Closing-the-Charging-Case/)
- [Part 10. Conclusion](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-10.-Conclusion/)