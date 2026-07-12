---
title: "AirPods and the Apple Unified Log. Part 6. Media pausing event"
layout: post
categories: [iOS, AirPods]
description: Investigating the Apple Unified Logging to find artefacts of connected Airpods. Part 6.
image:
  path: /assets/2026/airpods/airpods_6.png
  alt: FindMy
---

This post is the sixth in a serie that examines which AirPods related artefacts are recorded in the Apple Unified Log. This post focuses on pausing the music on the iPhone using a touch command on the left AirPod. 

## Pausing media
The pause command send from the Airpods. The log shows that a pause command was received, but it does not identify which AirPod generated the gesture.

Before that tree log of the `MediaRemote` subsystem where recorded.

```
2026-06-29 16:19:13.256498+0200 0xc28      Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.Remote] CoP: 0, call related session:0 delayPlay:0

2026-06-29 16:19:13.257176+0200 0x1277b    Default     0x0                  1944   0    BTAvrcp: (MediaRemote) [com.apple.amp.mediaremote:rr] Request: Command = <Pause>, SenderDevice = <iPhone>, SenderBundleIdentifier = <com.apple.BTAvrcp>, SenderPID = <1944>, commandID = <91AFB7CA-DEC3-4CAF-B6F3-39B5E34C8B78>, remote control interface = <com.apple.AVRCP>, appOptions = <0>, options = <{
    kMRMediaRemoteOptionSourceID = "B8:81:FA:11:FF:AB";
}><(null)> for 〖 􂞷 ❯ 􁐜 ❯ 􀭉 〗

2026-06-29 16:19:13.257237+0200 0x1277b    Default     0x0                  1944   0    BTAvrcp: (MediaRemote) [com.apple.amp.mediaremote:rr] Request: resolvePlayerPath<34DD2C1B-A241-4AA8-A5FD-ED049F33CC49> for 〖 􂞷 ❯ 􁐜 ❯ 􀭉 〗

2026-06-29 16:19:13.257306+0200 0x1277b    Default     0x0                  1944   0    BTAvrcp: (MediaRemote) [com.apple.amp.mediaremote:rr] Response: resolvePlayerPath<34DD2C1B-A241-4AA8-A5FD-ED049F33CC49> returned <<private>> for 〖 􂞷 ❯ 􁐜 ❯ 􀭉 〗 in 0.0001 seconds

2026-06-29 16:19:13.258178+0200 0x3cb98    Default     0x0                  1540   0    mediaremoted: [com.apple.amp.mediaremote:RemoteControl] Received command from client <MRDMediaRemoteClient 0x957459500, bundleIdentifier = com.apple.BTAvrcp, pid = 1944, entitlements=512>: <MRDMutableRemoteControlCommand 0x957451b60, command = Pause, SenderDevice = <iPhone>, SenderBundleIdentifier = <com.apple.BTAvrcp>, SenderPID = <1944>
, commandID = 91AFB7CA-DEC3-4CAF-B6F3-39B5E34C8B78
, remote control interface = com.apple.AVRCP
, appOptions = 0
, path = 【 􁊸 LOCL (iPhone) ❯ 􀐥 com.soundcloud.TouchApp (4514) SoundCloud ❯ 􀽍 default 】
, unresolvedPath = 〖 􂞷 ❯ 􁐜 ❯ 􀭉 〗
, nowPlayingAppStackOperation=NO
>
```

The stem click gesture was identified as `Pause`.
```
2026-06-29 16:19:13.258372+0200 0xeda8     Default     0x0                  1539   0    audioaccessoryd: (CoreUtils) [com.apple.bluetooth:BTSmartRoutingDaemon] Stem click gesture notification received, wx B8:81:FA:11:FF:AB cmd Pause

[...]

2026-06-29 16:19:13.259338+0200 0x3cb9f    Default     0x0                  1540   0    mediaremoted: [com.apple.amp.mediaremote:RemoteControl] Sending command <MRDMutableRemoteControlCommand 0x957452be0, command = Pause, SenderDevice = <iPhone>, SenderBundleIdentifier = <com.apple.BTAvrcp>, SenderPID = <1944>
, commandID = 91AFB7CA-DEC3-4CAF-B6F3-39B5E34C8B78
, remote control interface = com.apple.AVRCP
, appOptions = 0
, path = 【 􁊸 LOCL (iPhone) ❯ 􀐥 com.soundcloud.TouchApp (4514) SoundCloud ❯ 􀽍 default 】
, unresolvedPath = 〖 􂞷 ❯ 􁐜 ❯ 􀭉 〗
, nowPlayingAppStackOperation=NO
> to com.soundcloud.TouchApp (com.soundcloud.TouchApp-4514).
```

## Overview of the articles
- [Part 1. Introduction and methodology](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-1.-Introduction-and-methodology/)
- [Part 2. Opening the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-2.-Opening-the-Charging-Case/)
- [Part 3. Detecting AirPods removed and inserted into the ear](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-3.-Detecting-AirPods-removed-and-inserted-into-the-ear/)
- [Part 4. Closing the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-4.-Closing-the-Charging-Case/)
- [Part 5. Media play event](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-5.-Media-play-event/)
- Part 6. Media pause events
- [Part 7. Detecting AirPods removed from the ears](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-7.-Detecting-AirPods-removed-from-the-ears/)
- [Part 8. Detecting AirPods returned to the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-8.-Detecting-AirPods-returned-to-the-Charging-Case/)
- [Part 9. Closing the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-9.-Closing-the-Charging-Case/)
- [Part 10. Conclusion](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-10.-Conclusion/)