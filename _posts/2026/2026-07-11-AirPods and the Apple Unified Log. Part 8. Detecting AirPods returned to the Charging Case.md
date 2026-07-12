---
title: "AirPods and the Apple Unified Log. Part 8. Detecting AirPods returned to the Charging Case"
layout: post
categories: [iOS, AirPods]
description: Investigating the Apple Unified Logging to find artefacts of connected Airpods. Part 8.
image:
  path: /assets/2026/airpods/airpods_8.png
  alt: FindMy
---

This post is the eight in a serie that examines which AirPods related artefacts are recorded in the Apple Unified Log. This post focuses on returning the AirPods into the Charging Case

## Opening Charging case
When the Charging Case is openen it is recorded as powersource.

```
2026-06-29 16:19:16.653870+0200 0x3cc6a    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AADevicePowerSources] Power source unpublish for battery: Case
```

## Right AirPod in Charging Case
The first recorded log is from `Server.AACP`. This records that the primary bud is in the case while the secondary is recorded as out of case.
```
2026-06-29 16:19:17.562796+0200 0x3ce62    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.AACP] received in-ear state update from device B8:81:FA:11:FF:AB primary bud state = in case, secondary bud state = out of ear
```

This is followed by the same observation by `CUBluetoothClient`.
```
2026-06-29 16:19:17.574295+0200 0x69d      Default     0x0                  74     0    sharingd: (CoreUtils) [com.apple.CoreUtils:CUBluetoothClient] [Sharing-CA] Placement changed: CUBluetoothDevice B8:81:FA:11:FF:AB, ID 'f4cf7039-6e99-f1d6-df03-77a730fb317b', 'AirPods Pro', Mfg 'Apple Inc', Md A1602, PID 0x200E, VrI 0xD415, DT 20, FV '6F21', CnS 0x180019, PriP InCase, 2ndP OutOfEar, MagicPaired, AACP, Color 0 (White), DF 0xAB < ShareAudio AACP SoftwareVolume Tipi InEarDetect >, P OutOfEar -> InCase, S OutOfEar -> OutOfEar
```

## Left AirPod in Charging Case
For the left AirPod the same log sequence as for the right Airpod is recorded.
```
2026-06-29 16:19:17.672559+0200 0x3cc4c    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.AACP] received in-ear state update from device B8:81:FA:11:FF:AB primary bud state = in case, secondary bud state = in case
```
```
2026-06-29 16:19:17.680145+0200 0x69d      Default     0x0                  74     0    sharingd: (CoreUtils) [com.apple.CoreUtils:CUBluetoothClient] [Sharing-CA] Placement changed: CUBluetoothDevice B8:81:FA:11:FF:AB, ID 'f4cf7039-6e99-f1d6-df03-77a730fb317b', 'AirPods Pro', Mfg 'Apple Inc', Md A1602, PID 0x200E, VrI 0xD415, DT 20, FV '6F21', CnS 0x180019, PriP InCase, 2ndP InCase, MagicPaired, AACP, Color 0 (White), DF 0xAB < ShareAudio AACP SoftwareVolume Tipi InEarDetect >, P InCase -> InCase, S OutOfEar -> InCase
```

## Both AirPods in Charging Case
When both Airpods where place in the Charging Case, `SRDiscoveredDevice` recorded the setting  `nearbyInCase [...] no -> yes`
```
2026-06-29 16:19:18.791014+0200 0x3cef7    Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.bluetooth:SRDiscoveredDevice] Setting nearbyInCase B8:81:FA:11:FF:AB no -> yes
```

## Overview of the articles
- [Part 1. Introduction and methodology](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-1.-Introduction-and-methodology/)
- [Part 2. Opening the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-2.-Opening-the-Charging-Case/)
- [Part 3. Detecting AirPods removed and inserted into the ear](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-3.-Detecting-AirPods-removed-and-inserted-into-the-ear/)
- [Part 4. Closing the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-4.-Closing-the-Charging-Case/)
- [Part 5. Media play event](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-5.-Media-play-event/)
- [Part 6. Media pause events](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-6.-Media-pausing-event/)
- [Part 7. Detecting AirPods removed from the ears](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-7.-Detecting-AirPods-removed-from-the-ears/)
- Part 8. Detecting AirPods returned to the Charging Case
- [Part 9. Closing the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-9.-Closing-the-Charging-Case/)
- [Part 10. Conclusion](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-10.-Conclusion/)