---
title: "AirPods and the Apple Unified Log. Part 3. Detecting AirPods removed and inserted into the ear"
layout: post
categories: [iOS, AirPods]
description: Investigating the Apple Unified Logging to find artefacts of connected Airpods. Part 3.
image:
  path: /assets/2026/airpods/airpods_3.png
  alt: FindMy
---

This post is the third in a serie that examines which AirPods related artefacts are recorded in the Apple Unified Log. This post focuses on removing the AirPods from the Charging Case and inserting them into the ears.

The logs are presented in per Airpod, in chronological order. 

## Removing the AirPods from the Charging Case
Removing the AirPods causes the `nearbyInCase` state to change from `yes` to `no`.
```
2026-06-29 16:19:02.156028+0200 0x3cb9d    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.bluetooth:SRDiscoveredDevice] Setting nearbyInCase B8:81:FA:11:FF:AB yes -> no
```

## Right AirPod
### Right AirPod removed
First the primary AirPod was removed from the case. In the first logs, the right AirPod was identified as the primary bud.
```
2026-06-29 16:19:01.602565+0200 0x3cb78    Default     0x0                  229    0    searchpartyd: [com.apple.icloud.searchpartyd:classicPairing] Attributes changed: <Device [Identifier<Device> F4CF7039-6E99-F1D6-DF03-77A730FB317B]: <b881fa70ff0b type: .public> name:AirPods Pro model:AirPodsPro1,1 rssi:.rssi(-37) manufacturerData:<BluetoothManufacturerAdvertisementData payloadType:proximityPairing subType:status rawPayload:010e2004f99931000000dfff61cabfd770ff0b0010b113a9e3>
timestamp:2026-06-29T16:19:01.604+02:00>

2026-06-29 16:19:01.602660+0200 0x3cb9d    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AADevicePowerSources] Power source creating id for battery: Right

2026-06-29 16:19:01.602959+0200 0x3cb9d    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AADevicePowerSources] Power source publish update for battery: 'Right' with details:
{
    "Max Capacity" : 100,
    "Low Warn Level" : 20,
    "Is Present" : true,
    "Is Charged" : 0,
    "Name" : "AirPods Pro",
    "Type" : "Accessory Source",
    "Accessory Identifier" : "F4CF7039-6E99-F1D6-DF03-77A730FB317B",
    "Accessory Category" : "Headset",
    "Part Name" : "AirPods Pro 🅡",
    "Part Identifier" : "Right",
    "Is Charging" : true,
    "Time to Full Charge" : 0,
    "Transport Type" : "Bluetooth LE",
    "Current Capacity" : 95,
    "Vendor ID Source" : 1,
    "Optimized Battery Charging Engaged" : false,
    "Vendor ID" : 76,
    "Dynamic End of Charging Engaged" : false,
    "Group Identifier" : "F4CF7039-6E99-F1D6-DF03-77A730FB317B",
    "Product ID" : 8206,
    "Power Source State" : "AC Power",
}

2026-06-29 16:19:01.603508+0200 0x3cb9d    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AADevicePowerSources] Published: AADevicePowerSource, Batteries: C, R, 
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

Right: {
    "Max Capacity" : 100,
    "Low Warn Level" : 20,
    "Is Present" : true,
    "Is Charged" : 0,
    <…>
```

### Right AirPod out of ear
After removing the right AirPod from the Charging Case the log records it as out of ear: `Unknown -> OutOfEar`.

```
2026-06-29 16:19:01.594772+0200 0x3cba0    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.bluetooth:BTSmartRoutingDaemon] Wx Device found/updated: SFBLEDevice ID bb004e46-49fc-1000-8000-001ff3fb80df, BDA B8:81:FA:11:FF:AB, RSSI -37 (0)t~U, Ch 38, AdvD <4c000719010e2004f99931000000dfff61cabfd770ff0b0010b113a9e3>, ST Status, Nm 'BBVoyThp', Md 'BBYdRzGf', Paired yes, Cnx no, ST=uC?rPbo1?s, CC=0, OBC=Off, Batt C -97%; R +95%, headphone Status 0x102, bud is Primary/Right, source device count: 0 audio state: Idle lastConnect: cabfd7, tipiScore1: Unknown, tipiScore2: Unknown, idle time: 0, outofCaseTime Less than 5s, icloud Signed in YES usb no

2026-06-29 16:19:01.595011+0200 0x3cba0    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.bluetooth:SRDiscoveredDevice] Setting nearbyInEar B8:81:FA:11:FF:AB Unknown -> OutOfEar

2026-06-29 16:19:01.595016+0200 0x3cba0    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.bluetooth:SRDiscoveredDevice] Setting inEarStateUnified B8:81:FA:11:FF:AB Unknown -> OutOfEar
```

The log records the primary AirPod as `OutOfEar`, while the secondary AirPod remains `Unknown`. This is consistent with only the right AirPod having been removed from the Charging Case at that moment.

```
2026-06-29 16:19:03.326208+0200 0x69d      Default     0x0                  74     0    sharingd: (CoreUtils) [com.apple.CoreUtils:CUBluetoothClient] [Sharing-CA] Placement changed: CUBluetoothDevice B8:81:FA:11:FF:AB, ID 'f4cf7039-6e99-f1d6-df03-77a730fb317b', 'AirPods Pro', Mfg 'Apple Inc', Md A1602, PID 0x200E, VrI 0xD415, DT 20, FV '6F21', CnS 0x180019, PriP OutOfEar, MagicPaired, AACP, Color 0 (White), DF 0xAB < ShareAudio AACP SoftwareVolume Tipi InEarDetect >, P Unknown -> OutOfEar, S Unknown -> Unknown
```

```
2026-06-29 16:19:03.406422+0200 0x3cabb    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.AACP] received in-ear state update from device B8:81:FA:11:FF:AB primary bud state = out of ear, secondary bud state = unknown
```

### Right AirPod inserted in ear
When the primary right AirPod was placed in the ear the in ear status was logged.
```
2026-06-29 16:19:04.220269+0200 0x3c926    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.AACP] received in-ear state update from device B8:81:FA:11:FF:AB primary bud state = in ear, secondary bud state = out of ear
```

```
2026-06-29 16:19:04.239432+0200 0x3cb02    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.bluetooth:BTSmartRoutingDaemon] SmartRouting CONNECTED STATE shows inEar: yes for device B8:81:FA:11:FF:AB primary:InEar secondary:OutOfEar
```

```
2026-06-29 16:19:04.245906+0200 0x69d      Default     0x0                  74     0    sharingd: (CoreUtils) [com.apple.CoreUtils:CUBluetoothClient] [Sharing-CA] Placement changed: CUBluetoothDevice B8:81:FA:11:FF:AB, ID 'f4cf7039-6e99-f1d6-df03-77a730fb317b', 'AirPods Pro', Mfg 'Apple Inc', Md A1602, PID 0x200E, VrI 0xD415, DT 20, FV '6F21', CnS 0x180019, PriP InEar, 2ndP OutOfEar, MagicPaired, AACP, Color 0 (White), DF 0xAB < ShareAudio AACP SoftwareVolume Tipi InEarDetect >, P OutOfEar -> InEar, S OutOfEar -> OutOfEar
```

### Single bud
When only the right AirPod was out of ear the state of the "buds" was recorded as "SingleBud".

```
2026-06-29 16:19:04.249462+0200 0x3cc46    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AAAccessoryUsageSummaryManager] UpdateInEarState: OldBudState Invalid newBudState SingleBud secondsSinceOldBudState 96s singleBudDuration 10m bothBudsDuration 0m force no
```

## Left AirPod
### Left AirPod removed
When the left was removed from the Charging Case the individual battery entry was recorded as `Combined`.

```
2026-06-29 16:19:01.733334+0200 0x3caed    Default     0x0                  48     8    powerd: [com.apple.powerd:battery] Posted notifications for loss of power source id 5018

2026-06-29 16:19:01.733346+0200 0x3cba0    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AADevicePowerSources] Power source clear details for battery: Right

2026-06-29 16:19:01.733415+0200 0x3cba0    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AADevicePowerSources] Power source creating id for battery: Combined

2026-06-29 16:19:01.733460+0200 0x3cb3c    Default     0x0                  48     8    powerd: [com.apple.powerd:battery] Created new power source id 5019 for pid 1539

2026-06-29 16:19:01.733532+0200 0x3cba0    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AADevicePowerSources] Power source publish update for battery: 'Combined' with details:
{
    "Max Capacity" : 100,
    "Combined Parts" : 
    [
        {
            "Max Capacity" : 100,
            "Low Warn Level" : 20,
            "Is Present" : true,
            "Is Charged" : 1,
            "Name" : "AirPods Pro",
            "Type" : "Accessory Source",
            "Accessory Identifier" : "F4CF7039-6E99-F1D6-DF03-77A730FB317B",
            "Accessory Category" : "Headset",
            "Part Name" : "AirPods Pro 🅛",
            "Part Identifier" : "Left",
            "Is Charging" : true,
            "Time to Full Charge" : 0,
            "Transport Type" : "Bluetooth LE",
            "Current Capacity" : 100,
            "Vendor ID Source" : 1,
            "Optimized Battery Charging Engaged" : false,
            "Vendor ID" : 76,
            "Dynamic End of Charging Engaged" : false,
            "Group Identifier" : "F4CF7039-6E99-F1D6-DF03-77A730FB317B",
            "Product ID" : 8206,
            "Power S<…>

2026-06-29 16:19:01.733550+0200 0x3cba0    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AADevicePowerSources] Is Charged" : 0,
    "Name" : "AirPods Pro",
    "Type" : "Accessory Source",
    "Accessory Identifier" : "F4CF7039-6E99-F1D6-DF03-77A730FB317B",
    "Accessory Category" : "Headset",
    "Part Identifier" : "Combined",
    "Is Charging" : true,
    "Time to Full Charge" : 0,
    "Transport Type" : "Bluetooth LE",
    "Current Capacity" : 95,
    "Vendor ID Source" : 1,
    "Optimized Battery Charging Engaged" : false,
    "Vendor ID" : 76,
    "Dynamic End of Charging Engaged" : false,
    "Group Identifier" : "F4CF7039-6E99-F1D6-DF03-77A730FB317B",
    "Product ID" : 8206,
    "Power Source State" : "AC Power",
}
```

### Left AirPod out of ear
The log records the secondary AirPod as `OutOfEar`. This is consistent with the left AirPod being removed from the Charging Case at that moment.

The `BTSmartRoutingDaemon` changed the values from unknown to out of ear.
```
2026-06-29 16:19:03.253462+0200 0x3cba1    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.bluetooth:BTSmartRoutingDaemon] SmartRouting CONNECTED STATE shows inEar: no for device B8:81:FA:11:FF:AB primary:OutOfEar secondary:Unknown

2026-06-29 16:19:03.253470+0200 0x3cba1    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.bluetooth:SRDiscoveredDevice] Setting aacpInEarState B8:81:FA:11:FF:AB Unknown -> OutOfEar
```

```
2026-06-29 16:19:03.930051+0200 0x69d      Default     0x0                  74     0    sharingd: (CoreUtils) [com.apple.CoreUtils:CUBluetoothClient] [Sharing-CA] Placement changed: CUBluetoothDevice B8:81:FA:11:FF:AB, ID 'f4cf7039-6e99-f1d6-df03-77a730fb317b', 'AirPods Pro', Mfg 'Apple Inc', Md A1602, PID 0x200E, VrI 0xD415, DT 20, FV '6F21', CnS 0x180019, PriP OutOfEar, 2ndP OutOfEar, MagicPaired, AACP, Color 0 (White), DF 0xAB < ShareAudio AACP SoftwareVolume Tipi InEarDetect >, P OutOfEar -> OutOfEar, S Unknown -> OutOfEar
```

After removing the left secondary AirPod from the Charging Case it was as out of ear.
```
2026-06-29 16:19:04.220269+0200 0x3c926    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.AACP] received in-ear state update from device B8:81:FA:11:FF:AB primary bud state = in ear, secondary bud state = out of ear
```

### Left AirPod in ear
When the left AirPod was inserted the following state transition was logged.
```
2026-06-29 16:19:04.474109+0200 0x3c926    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.AACP] received in-ear state update from device B8:81:FA:11:FF:AB primary bud state = in ear, secondary bud state = in ear
```

Both AirPods where then logged with status in ear.

```
2026-06-29 16:19:04.485725+0200 0x69d      Default     0x0                  74     0    sharingd: (CoreUtils) [com.apple.CoreUtils:CUBluetoothClient] [Sharing-CA] Placement changed: CUBluetoothDevice B8:81:FA:11:FF:AB, ID 'f4cf7039-6e99-f1d6-df03-77a730fb317b', 'AirPods Pro', Mfg 'Apple Inc', Md A1602, PID 0x200E, VrI 0xD415, DT 20, FV '6F21', CnS 0x180019, PriP InEar, 2ndP InEar, MagicPaired, AACP, Color 0 (White), DF 0xAB < ShareAudio AACP SoftwareVolume Tipi InEarDetect >, P InEar -> InEar, S OutOfEar -> InEar
```

```
2026-06-29 16:19:04.493310+0200 0x3cb9d    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.bluetooth:BTSmartRoutingDaemon] SmartRouting CONNECTED STATE shows inEar: yes for device B8:81:FA:11:FF:AB primary:InEar secondary:InEar
```

### Both buds
Also the `SingleBud` status changed to  `BothBuds`.
```
2026-06-29 16:19:04.494054+0200 0x3ca40    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AAAccessoryUsageSummaryManager] UpdateInEarState: OldBudState SingleBud newBudState BothBuds secondsSinceOldBudState 0s singleBudDuration 10m bothBudsDuration 0m force no
```

And both AirPods where logged as connected.
```
2026-06-29 16:19:05.062885+0200 0x3cb83    Default     0x0                  145    0    findmydeviced: [com.apple.icloud.findmydeviced:_] FMDAudioChannelStatus right dictionaryValueForConnectedState: _playingSound=0 connected=1 -> playing=0

2026-06-29 16:19:05.062891+0200 0x3cb83    Default     0x0                  145    0    findmydeviced: [com.apple.icloud.findmydeviced:_] FMDAudioChannelStatus left dictionaryValueForConnectedState: _playingSound=0 connected=1 -> playing=0
```

## Overview of the articles
- [Part 1. Introduction and methodology](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-1.-Introduction-and-methodology/)
- [Part 2. Opening the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-2.-Opening-the-Charging-Case/)
- Part 3. Detecting AirPods removed and inserted into the ear
- [Part 4. Closing the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-4.-Closing-the-Charging-Case/)
- [Part 5. Media play event](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-5.-Media-play-event/)
- [Part 6. Media pause events](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-6.-Media-pausing-event/)
- [Part 7. Detecting AirPods removed from the ears](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-7.-Detecting-AirPods-removed-from-the-ears/)
- [Part 8. Detecting AirPods returned to the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-8.-Detecting-AirPods-returned-to-the-Charging-Case/)
- [Part 9. Closing the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-9.-Closing-the-Charging-Case/)
- [Part 10. Conclusion](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-10.-Conclusion/)