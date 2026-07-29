---
title: "Apple Unified Log artifacts of changing AirPods listening mode using an iPhone"
layout: post
categories: [iOS, AirPods]
description: Investigating the Apple Unified Logging to find artefacts of changing the state of the Active Noise Canceling using the iPhone.
image:
  path: /assets/2026/listeningmode/iphone.png
  alt: anc
---

AirPods listening modes can be changed either from the AirPods themselves or from Control Center on a paired iPhone. Although both methods produce the same result from a user perspective, they may generate different artifacts in the Apple Unified Log.

This article examines the logging generated when the listening mode is changed from the iPhone. A companion article examines the same action performed directly [on the AirPods](https://www.mennovanveenendaal.com/posts/Apple-Unified-Log-artifacts-of-changing-AirPods-listening-mode-using-AirPods/).

![listening_modes](/assets/2026/listeningmode/listening_modes.png){: w="200" h="400" }
_Fig.1 Listening modes_

## Test procedure
To generate the logging I performed the following actions using a AirPods Pro with firmware 6F21 and a paired iPhone 12 using iOS 26.5. The Airpods where placed in ear with the listening mode set to "Noise Cancelation". No audio was playing while the listening modes were changed.

1. On phone select the additional controls in the [Control Center](https://support.apple.com/en-us/108918).
2. Tap Off
3. Tap Transparency
4. Tap Noise Cancellation

The Apple Unified Log was collected filtering between the times of actions.

## "Off" listening mode
```
2026-07-17 19:50:02.077617+0200 0x623      Default     0xd1310              35     0    SpringBoard: (MediaControls) [com.apple.amp.mediacontrols:Volume] MRUListeningModeController set bluetooth to listening mode: AVOutputDeviceBluetoothListeningModeNormal | AVOutputDeviceBluetoothListeningModeActiveNoiseCancellation | device: <MRAVConcreteOutputDevice:0xb06c84e00 (local) "AirPods Pro" uid="B8:81:FA:11:FF:AB" group_id="12E582F3-C755-452A-93CA-F5E5C802AA4C" bluetooth_id=(null) type=Bluetooth subtype=Headphones clusterType=None group-leader group-contains-leader parent-group-discoverable-leader enc-prog-dl-assets fetch-sender-media-data opt-audio-ui BTHeadphones76,8206 bluetooth-sharing (batt l:1.00|r:1.00) AirPlay infra>
```
In this part of the AUL, the first log mentioning the AirPods is from the `SpringBoard` process, `MediaControls` / `Volume` subsystem. It logs that the `MRUListeningModeController` set bluetooth to listening mode: `AVOutputDeviceBluetoothListeningModeNormal`. The device is logged as `AirPods Pro` with uid=`B8:81:FA:11:FF:AB`.

Throughout the sequence the Bluetooth identifier `B8:81:FA:11:FF:AB` remains constant, allowing the log entries from different processes to be correlated to the same AirPods Pro device.

Records from the `com.apple.BackBoard` `TouchEvents` subsystem precede the state transition. Although these records correspond to screen interaction, no evidence was found that directly associates a specific touch event with the listening mode change.

```
2026-07-17 19:50:02.077934+0200 0x623      Default     0xd1310              35     0    SpringBoard: (AVRouting) [com.apple.coremedia:] <<<< AVOutputDevice >>>> -[AVOutputDevice setCurrentBluetoothListeningMode:error:]: called (mode=AVOutputDeviceBluetoothListeningModeNormal)
```

```
2026-07-17 19:50:02.079785+0200 0x623      Default     0xd1310              35     0    SpringBoard: (AVRouting) [com.apple.coremedia:] <<<< AVOutputDevice (FigRouteDescriptor) >>>> -[AVFigRouteDescriptorOutputDeviceImpl setCurrentBluetoothListeningMode:error:]: Grabbing endpoint for route descriptor <private>
```

```
2026-07-17 19:50:02.081832+0200 0x623      Default     0xd1310              35     0    SpringBoard: (AVRouting) [com.apple.coremedia:] <<<< AVOutputDevice (FigRouteDescriptor) >>>> -[AVFigRouteDescriptorOutputDeviceImpl setCurrentBluetoothListeningMode:error:]: Setting kFigEndpointProperty_ListeningMode to 1
```
The next three records are also from the `SpringBoard` process. 
First `mode=AVOutputDeviceBluetoothListeningModeNormal` is called. Then the endpoint is grabbed for a route descriptor. And lastly the `kFigEndpointProperty_ListeningMode` is set to 1.

Apple [documents](https://developer.apple.com/documentation/sensorkit/srheadphonesettings/listeningmode-swift.enum) the `ListeningMode` enumeration as containing four possible values. During testing, only three listening modes were available because of the [capabilities](https://support.apple.com/en-gb/guide/airpods/dev9812f5cc3/web) of the AirPods Pro used in this experiment.

```
2026-07-17 19:50:02.082202+0200 0x48895    Default     0xd1310              110    0    audiomxd: (VirtualAudio) [com.apple.coreaudio:va_device] Device_Bluetooth_Aspen.cpp:762   Bluetooth audio device with UID "B8:81:FA:11:FF:AB-btaudio": Setting Bluetooth property lstm.
```
`audiomxd` logged that its setting the `lstm` (or listening mode) for device `with UID "B8:81:FA:11:FF:AB-btaudio"`.

```
2026-07-17 19:50:02.082261+0200 0x48895    Default     0xd1310              110    0    audiomxd: (BTAudioHALPlugin) [com.apple.bluetooth:BTAudio] Accessory mListenMode set to 1 for mAudioObjectID 58
```

```
2026-07-17 19:50:02.082264+0200 0x48895    Default     0xd1310              110    0    audiomxd: (BTAudioHALPlugin) [com.apple.bluetooth:BTAudio] Setting listen mode  1
```
`audiomxd` then reports that the accessory listen mode is set to `1` for AudioObject with ID 58.

```
2026-07-17 19:50:02.082332+0200 0x48895    Default     0xd1310              110    0    audiomxd: (MediaExperience) [com.apple.coremedia:] -CMVAEndpoint- vaeSetBluetoothListeningMode: Called to set bluetooth listening mode to Normal
```
`audiomxd` logs a call to set the listening mode to `Normal`.

```
2026-07-17 19:50:02.082477+0200 0x487b1    Default     0xd1310              97     0    bluetoothd: [com.apple.bluetooth:Server.Core] Listening Mode is set to 1 for Device B8:81:FA:11:FF:AB
```
Followed by a logline from the `bluetoothd` process logging that the listening mode is set to `1` for `Device B8:81:FA:11:FF:AB`. This UID can be correlated to the paired `AirPods Pro`. 

```
2026-07-17 19:50:02.082505+0200 0x4878a    Default     0xd1310              110    0    audiomxd: (VirtualAudio) [com.apple.coreaudio:va_device] Device_Bluetooth_Aspen.cpp:1044  Received notification (lstm) from bluetooth audio device with UID "B8:81:FA:11:FF:AB-btaudio"
```

```
2026-07-17 19:50:02.082579+0200 0x4878a    Default     0xd1310              110    0    audiomxd: (VirtualAudio) [com.apple.coreaudio:va_device] Device_Bluetooth_Aspen.cpp:925   Audio listen mode changed to 1 for bluetooth audio device with UID "B8:81:FA:11:FF:AB-btaudio"
```
To complete the change, `audiomxd` logs that a notification is received (listening mode) from an audio device with the same UID `B8:81:FA:11:FF:AB`. The audio listen mode for this device has changed to `1`. 

##  Transparent listening mode
Changing the listening mode from "Off" to "Transparency" produces the same sequence of logging events. The primary difference is the value associated with the selected listening mode.

```
2026-07-17 19:50:03.785003+0200 0x623      Default     0xd1318              35     0    SpringBoard: (MediaControls) [com.apple.amp.mediacontrols:Volume] MRUListeningModeController set bluetooth to listening mode: AVOutputDeviceBluetoothListeningModeAudioTransparency | AVOutputDeviceBluetoothListeningModeNormal | device: <MRAVConcreteOutputDevice:0xb06c6b9c0 (local) "AirPods Pro" uid="B8:81:FA:11:FF:AB" group_id="12E582F3-C755-452A-93CA-F5E5C802AA4C" bluetooth_id=(null) type=Bluetooth subtype=Headphones clusterType=None group-leader group-contains-leader parent-group-discoverable-leader enc-prog-dl-assets fetch-sender-media-data opt-audio-ui BTHeadphones76,8206 bluetooth-sharing (batt l:1.00|r:1.00) AirPlay infra>
```
The `SpringBoard` process, `MediaControls` / `Volume` subsystem logs that the `MRUListeningModeController` set bluetooth to listening mode: `AVOutputDeviceBluetoothListeningModeAudioTransparency`. After the `|` is a state that is in accordance with the current listening mode, `AVOutputDeviceBluetoothListeningModeNormal`.

The AUL again showed many loglines logged by the `com.apple.BackBoard`proccesses `TouchEvents` subsystem. And again there where nono evidence was found that directly associates a specific touch event with the listening mode change.

```
2026-07-17 19:50:03.785153+0200 0x623      Default     0xd1318              35     0    SpringBoard: (AVRouting) [com.apple.coremedia:] <<<< AVOutputDevice >>>> -[AVOutputDevice setCurrentBluetoothListeningMode:error:]: called (mode=AVOutputDeviceBluetoothListeningModeAudioTransparency)
```

```
2026-07-17 19:50:03.785171+0200 0x623      Default     0xd1318              35     0    SpringBoard: (AVRouting) [com.apple.coremedia:] <<<< AVOutputDevice (FigRouteDescriptor) >>>> -[AVFigRouteDescriptorOutputDeviceImpl setCurrentBluetoothListeningMode:error:]: Grabbing endpoint for route descriptor <private>
```

```
2026-07-17 19:50:03.785802+0200 0x623      Default     0xd1318              35     0    SpringBoard: (AVRouting) [com.apple.coremedia:] <<<< AVOutputDevice (FigRouteDescriptor) >>>> -[AVFigRouteDescriptorOutputDeviceImpl setCurrentBluetoothListeningMode:error:]: Setting kFigEndpointProperty_ListeningMode to 3
```
As in the previous example, SpringBoard first records the requested listening mode before the request propagates to `audiomxd` and `bluetoothd`.

In this sequence `mode=AVOutputDeviceBluetoothListeningModeAudioTransparency` is called, the endpoint is grabbed for a route descriptor and the `kFigEndpointProperty_ListeningMode` is set to 3.

```
2026-07-17 19:50:03.786204+0200 0x4898a    Default     0xd1318              110    0    audiomxd: (VirtualAudio) [com.apple.coreaudio:va_device] Device_Bluetooth_Aspen.cpp:762   Bluetooth audio device with UID "B8:81:FA:11:FF:AB-btaudio": Setting Bluetooth property lstm.
```

```
2026-07-17 19:50:03.786266+0200 0x4898a    Default     0xd1318              110    0    audiomxd: (BTAudioHALPlugin) [com.apple.bluetooth:BTAudio] Accessory mListenMode set to 3 for mAudioObjectID 58
```

```
2026-07-17 19:50:03.786268+0200 0x4898a    Default     0xd1318              110    0    audiomxd: (BTAudioHALPlugin) [com.apple.bluetooth:BTAudio] Setting listen mode  3
```

```
2026-07-17 19:50:03.786341+0200 0x4898a    Default     0xd1318              110    0    audiomxd: (MediaExperience) [com.apple.coremedia:] -CMVAEndpoint- vaeSetBluetoothListeningMode: Called to set bluetooth listening mode to Transparency
```
The `audiomxd` logs setting the listening mode for the same device with UID `B8:81:FA:11:FF:AB-btaudio` to `3` and a call to set the listening mode to `Transparency`.

```
2026-07-17 19:50:03.786514+0200 0x48897    Default     0xd1318              110    0    audiomxd: (VirtualAudio) [com.apple.coreaudio:va_device] Device_Bluetooth_Aspen.cpp:1044  Received notification (lstm) from bluetooth audio device with UID "B8:81:FA:11:FF:AB-btaudio"
```
`audiomxd` records receiving a notification (listening mode).

```
2026-07-17 19:50:03.786547+0200 0x48456    Default     0xd1318              97     0    bluetoothd: [com.apple.bluetooth:Server.Core] Listening Mode is set to 3 for Device B8:81:FA:11:FF:AB
```

```
2026-07-17 19:50:03.786586+0200 0x48897    Default     0xd1318              110    0    audiomxd: (VirtualAudio) [com.apple.coreaudio:va_device] Device_Bluetooth_Aspen.cpp:925   Audio listen mode changed to 3 for bluetooth audio device with UID "B8:81:FA:11:FF:AB-btaudio"
```
And also in this sequence, the last loglines conclude that the audio listen mode is changed to `3` for the device.

## Conclusion
When the iPhone is used to change the state of the listening mode the first log entry directly related to the listening mode change is from the  `SpringBoard` process. `SpringBoard` is the process that, among others, manages the [home screen](https://theapplewiki.com/wiki/Filesystem:/System/Library/CoreServices/SpringBoard.app), app icons and handeling notifications. 

Following this log, `audiomxd` and `bluetoothd` start to log a change in the listening mode and a notification from the AirPods regarding the listening mode. 

This **indicates** that the state transition is initiated by the iPhone user interface before the request is forwarded to the Bluetooth subsystem.

This log sequence is in contrast with the sequence found when the [AirPods are used to switch the listening mode](https://www.mennovanveenendaal.com/posts/Apple-Unified-Log-artifacts-of-changing-AirPods-listening-mode-using-AirPods/):

|Changed via|First process|First evidence|Interpretation|
|---|---|---|---|
|iPhone|`SpringBoard`|`MRUListeningModeController`|Request originated from Control Center|
|AirPods|`bluetoothd`|`Received Listen mode (control cmd 0x0D)`|Request originated from the AirPods|