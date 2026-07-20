---
title: "AirPods and the Apple Unified Log. Part 5. Media play event"
layout: post
categories: [iOS, AirPods]
description: Investigating the Apple Unified Logging to find artefacts of connected Airpods. Part 5
image:
  path: /assets/2026/airpods/airpods_5.png
  alt: FindMy
---

This post is the fifth in a serie that examines which AirPods related artefacts are recorded in the Apple Unified Log. This post focuses on starting the music on the iPhone using a touch command on the right AirPod. 

## Starting media
The log shows that a play command was received. Based on the available logging, the gesture can be attributed to the paired AirPods, but not to an individual left or right AirPod.

```
2026-06-29 16:19:06.688050+0200 0xc28      Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.Remote] Received AVRCP Play command from device B8:81:FA:11:FF:AB

2026-06-29 16:19:06.688054+0200 0xc28      Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.Remote] CoP: 0, call related session:0 delayPlay:0

2026-06-29 16:19:06.688507+0200 0x1277b    Default     0x0                  1944   0    BTAvrcp: (MediaRemote) [com.apple.amp.mediaremote:RemoteControl] [MRSendCommand] WHAPRO: Sending Play Command with Timestamp: Mon Jun 29 16:19:06 2026
```

## Play command
Right after the play command, the command to start Soundcloud was logged. Before that tree log of the `MediaRemote` subsystem where recorded.

```
2026-06-29 16:19:06.688681+0200 0x1277b    Default     0x0                  1944   0    BTAvrcp: (MediaRemote) [com.apple.amp.mediaremote:rr] Request: Command = <Play>, SenderDevice = <iPhone>, SenderBundleIdentifier = <com.apple.BTAvrcp>, SenderPID = <1944>, commandID = <911A6537-F558-4687-8C73-79C7B333866C>, remote control interface = <com.apple.AVRCP>, appOptions = <0>, options = <{
    kMRMediaRemoteOptionSourceID = "B8:81:FA:11:FF:AB";
}><(null)> for 〖 􂞷 ❯ 􁐜 ❯ 􀭉 〗
```
```
2026-06-29 16:19:06.688737+0200 0x1277b    Default     0x0                  1944   0    BTAvrcp: (MediaRemote) [com.apple.amp.mediaremote:rr] Request: resolvePlayerPath<D27862B1-4B3A-442F-A8BF-FAD1052BB61C> for 〖 􂞷 ❯ 􁐜 ❯ 􀭉 〗
```
```
2026-06-29 16:19:06.688788+0200 0x1277b    Default     0x0                  1944   0    BTAvrcp: (MediaRemote) [com.apple.amp.mediaremote:rr] Response: resolvePlayerPath<D27862B1-4B3A-442F-A8BF-FAD1052BB61C> returned <<private>> for 〖 􂞷 ❯ 􁐜 ❯ 􀭉 〗 in 0.0001 seconds

2026-06-29 16:19:06.689425+0200 0x3cbcd    Default     0x0                  1540   0    mediaremoted: [com.apple.amp.mediaremote:RemoteControl] Received command from client <MRDMediaRemoteClient 0x957459500, bundleIdentifier = com.apple.BTAvrcp, pid = 1944, entitlements=512>: <MRDMutableRemoteControlCommand 0x957452be0, command = Play, SenderDevice = <iPhone>, SenderBundleIdentifier = <com.apple.BTAvrcp>, SenderPID = <1944>
, commandID = 911A6537-F558-4687-8C73-79C7B333866C
, remote control interface = com.apple.AVRCP
, appOptions = 0
, path = 【 􁊸 LOCL (iPhone) ❯ 􀐥 com.soundcloud.TouchApp (4514) SoundCloud ❯ 􀽍 default 】
, unresolvedPath = 〖 􂞷 ❯ 􁐜 ❯ 􀭉 〗
, nowPlayingAppStackOperation=NO
>

[...]

2026-06-29 16:19:06.694194+0200 0x3caee    Default     0x0                  1540   0    mediaremoted: [com.apple.amp.mediaremote:rr] Update: sendRemoteControlCommand<911A6537-F558-4687-8C73-79C7B333866C> Sending nowPlayingAppStackPopEligible command...
2026-06-29 16:19:06.694555+0200 0x3caee    Default     0x0                  1540   0    mediaremoted: [com.apple.amp.mediaremote:RemoteControl] Sending command <MRDMutableRemoteControlCommand 0x957450720, command = Play, SenderDevice = <iPhone>, SenderBundleIdentifier = <com.apple.BTAvrcp>, SenderPID = <1944>
, commandID = 911A6537-F558-4687-8C73-79C7B333866C
, remote control interface = com.apple.AVRCP
, appOptions = 0
, path = 【 􁊸 LOCL (iPhone) ❯ 􀐥 com.soundcloud.TouchApp (4514) SoundCloud ❯ 􀽍 default 】
, unresolvedPath = 〖 􂞷 ❯ 􁐜 ❯ 􀭉 〗
, nowPlayingAppStackOperation=NO
> to com.soundcloud.TouchApp (com.soundcloud.TouchApp-4514).

[...]
```

The stem click gesture was identified as `Play`.
```
2026-06-29 16:19:06.689698+0200 0xeda8     Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.bluetooth:BTSmartRoutingDaemon] Stem click gesture notification received, wx B8:81:FA:11:FF:AB cmd Play
```

## Streaming music
Shortly after the play command is processed, `bluetoothd` records the establishment of an A2DP media stream to the AirPods.

```
2026-06-29 16:19:06.835389+0200 0x3cabb    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.A2DP] Starting Media connection to device B8:81:FA:11:FF:AB. Current stream state is 3 and pending stream request is 0

2026-06-29 16:19:06.835390+0200 0x3cabb    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.A2DP] Attempting to start streaming to device B8:81:FA:11:FF:AB

[...]

2026-06-29 16:19:07.033169+0200 0x3caa5    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.A2DP] Received confirm from B8:81:FA:11:FF:AB to start streaming

2026-06-29 16:19:07.033171+0200 0x3caa5    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.A2DP] Starting A2DP audio streaming to device B8:81:FA:11:FF:AB
```

During audio streaming, the AirPods ~~continue to report the active listening mode as `ANC Transparency`.~~ listening modus is logged as `LsnM Transparency`.

```
2026-06-29 16:19:07.047049+0200 0x3cc4b    Default     0x0                  97     0    bluetoothd: (CoreUtils) [com.apple.bluetooth:CBDaemonServer] Monitor Device Found: CBDevice F4CF7039-6E99-F1D6-DF03-77A730FB317B, BDA <private>, Nm <private> , Md AirPodsPro1,1, PID 0x200E (AirPodsPro1,1), PrNm AirPods Pro, VID 0x004C, VS 1, DsFl 0xA08000 < WxStatus Connections Pairing >, DvF 0x5081D80407F < AACP MagicPaired ShareAudio SoftwareVolume Tipi PSE AudioOwner ClassicPaired SpatialAudio AdvancedAppleAudio ANC Transparency SpatialAudioAllowed UTPConnected SmartRouting Connectable >, DvT Headphones, RSSI -38, CnS 0x980019 < HFP AVRCP A2DP AACP GATT ACL >, SupS 0x180019 < HFP AVRCP A2DP AACP GATT >, AStS A2DP, Freq 2.4, Ch 37, Battery L -100% R -95% C -97%, ClkH L NoiseManagement R NoiseManagement, ECC 2, MCCp 1, MCC 1, CVer'1.4.1', DbTp C Basic, GAPA 0x1 < Real >, FV '6F21', LsnM Transparency, LsMC 0x6 < ANC Transparency >, BTv 5.0, MicM Auto, Plcm P InEar S InEar M Enabled, Prim Right, modU A2084, SN '[...]', SN Left'[...]', SN Right '[...]', srMd Enabled, spAM ContentDriven, AdTsMC <[...]>, AMfD <4c 00 07 19 01 0e 20 0b a9 <…>
```

## Overview of the articles
- [Part 1. Introduction and methodology](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-1.-Introduction-and-methodology/)
- [Part 2. Opening the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-2.-Opening-the-Charging-Case/)
- [Part 3. Detecting AirPods removed and inserted into the ear](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-3.-Detecting-AirPods-removed-and-inserted-into-the-ear/)
- [Part 4. Closing the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-4.-Closing-the-Charging-Case/)
- Part 5. Media play event
- [Part 6. Media pause events](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-6.-Media-pausing-event/)
- [Part 7. Detecting AirPods removed from the ears](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-7.-Detecting-AirPods-removed-from-the-ears/)
- [Part 8. Detecting AirPods returned to the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-8.-Detecting-AirPods-returned-to-the-Charging-Case/)
- [Part 9. Closing the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-9.-Closing-the-Charging-Case/)
- [Part 10. Conclusion](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-10.-Conclusion/)