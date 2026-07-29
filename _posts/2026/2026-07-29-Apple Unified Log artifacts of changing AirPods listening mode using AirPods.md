---
title: "Apple Unified Log artifacts of changing AirPods listening mode using AirPods"
layout: post
categories: [iOS, AirPods]
description: Investigating the Apple Unified Logging to find artefacts of changing the state of the Active Noise Canceling using the AirPods.
image:
  path: /assets/2026/listeningmode/airpods.png
  alt: anc
---

AirPods listening modes can be changed either from the AirPods themselves or from Control Center on a paired iPhone. Although both methods produce the same result from a user perspective, they may generate different artifacts in the Apple Unified Log.

This article examines the logging generated when the listening mode is changed from the AirPods. A companion article examines the same action performed directly on the iPhone.

## Test procedure
To generate the logging I performed the following actions using a AirPods Pro with firmware 6F21 and a paired iPhone 12 using iOS 26.5. No audio was playing while the listening modes were changed.

1. Both AirPods placed in ear, Noise Cancelation on.
2. On right earbud long hold, AirPods to Transparant listening mode.
3. On left long hold, back to Noise Cancellation.

The Apple Unified Log was collected filtering between the times of these actions. The AUL was searched for logs to indicate from where the command to switch between modus was send. 

## Transparant listening mode

```
2026-07-17 16:58:02.902769+0200 0x35112    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.AACP] Received Listen mode (control cmd 0x0D) as Transparency from device B8:81:FA:11:FF:AB
```

```
2026-07-17 16:58:02.902771+0200 0x35112    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.AACP] Set ListeningMode device setting : ListeningMode value 3
```
The first observable records originate from `bluetoothd`.  The first line logs that the listening mode was received as `Transarancy` from device `B8:81:FA:11:FF:AB`.

The second line logs the setting of the `ListeningMode device setting` to `value 3`.

```
2026-07-17 16:58:02.902778+0200 0x35112    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.Core] Listening Mode is set to 3 for Device B8:81:FA:11:FF:AB
```
This is followed by a log from the `bluetoothd` that the Listening Mode for Device B8:81:FA:11:FF:AB is set to `3`.

```
2026-07-17 16:58:02.919665+0200 0x35211    Default     0x0                  110    0    audiomxd: (VirtualAudio) [com.apple.coreaudio:va_port  ]  Port_Bluetooth_Aspen.cpp:501   Notifying listeners: changed property lstm for port with name "AirPods Pro", UID "B8:81:FA:11:FF:AB-tsco", and type 'phpb'
```

`audiomxd` logged that the `listeners` where notified that the AirPods with UID `UID "B8:81:FA:11:FF:AB-tsco` had a changed property.

```
2026-07-17 16:58:02.919781+0200 0x352ba    Default     0x0                  110    0    audiomxd: (MediaExperience) [com.apple.coremedia:] -CMVAEndptMgr- cmsmBluetoothListeningModeListener_block_invoke: Listening mode for port=679 changed to Transparency
```

`audiomxd` then logs that the listening mode was changed to `Transparency`.

```
2026-07-17 16:58:02.926165+0200 0x3534c    Default     0x0                  80     0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AAServicesDaemon] reporting device:
```

```
2026-07-17 16:58:02.926174+0200 0x3534c    Default     0x0                  80     0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AAServicesDaemon] AADevice id: F4CF7039-6E99-F1D6-DF03-77A730FB317B, nm AirPods Pro, ml AirPodsPro1,1, pid 8206, 
color 255, 
AutoANC: cap -, cfg L, 
, comb LR Batt LR '-90.00%'[ls: 2026-07-17-14:56:09.241, sFL: 0x1 < HeadsetAdv >]Batt L '-93.00%'[ls: 2026-07-17-14:56:09.241, sFL: 0x1 < HeadsetAdv >], Batt R '-90.00%'[ls: 2026-07-17-14:56:09.241, sFL: 0x1 < HeadsetAdv >], 
Bbl: cap -, 
Cam ctl: cap -, 
Chr rem: cap -, DEOC: cap -, OBC: cap +, en +, 
HG: 
HA: cap -, v2 cap -, HP: cap -, PPE: cap -, HT: cap -, 
HR: cap -, 
Lsn: md Trnsp, cfg 0x6 < ANC Transparency >, 
Pl: md +, pr InEar, sc InEar, 
smRtS 0x4 < Routed >, temp mg -, 
SrMt: cap +, 
Sldt: cap -, 
SS: as Idle, fqBd 2.4, aos None, 
Misc Caps: cas snd -, ear fit -, ff upl -, hide er -, hide off -, ovd str -, pmee -, pt -, wrd lssls -, Misc Info: enh trn V2,lst conn '2026-07-17 14:56:09.000', 

CBDevice F4CF7039-6E99-F1D6-DF03-77A730FB317B, BDA <private>, Nm <private> , Md AirPodsPro1,1, PID 0x200E (AirPodsPro1,1), PrNm AirPods Pro, VID 0x004C, VS 1, DsFl 0xA08<…>
```
And the `audioaccessoryd` logs the AirPods with the listening mode as transparant: `Lsn: md Trnsp`.

## Set to Noise Cancellation
When switching to Noise Cancellation via the AirPods the same log sequence can be derived from the AUL.

```
2026-07-17 16:58:06.564160+0200 0x3532c    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.AACP] Received Listen mode (control cmd 0x0D) as ANC from device B8:81:FA:11:FF:AB
```
First it's logged that the `Listen mode` is received as `ANC`. 

```
2026-07-17 16:58:06.564162+0200 0x3532c    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.AACP] Set ListeningMode device setting : ListeningMode value 2
```

```
2026-07-17 16:58:06.564168+0200 0x3532c    Default     0x0                  97     0    bluetoothd: [com.apple.bluetooth:Server.Core] Listening Mode is set to 2 for Device B8:81:FA:11:FF:AB
```
Then the it's logged that the listening mode is set to `2` for device `B8:81:FA:11:FF:AB`.

```
2026-07-17 16:58:06.572689+0200 0x35211    Default     0x0                  110    0    audiomxd: (VirtualAudio) [com.apple.coreaudio:va_port  ]  Port_Bluetooth_Aspen.cpp:501   Notifying listeners: changed property lstm for port with name "AirPods Pro", UID "B8:81:FA:11:FF:AB-tacl", and type 'phpB'
```
Followed by a log that the listeners where notified that there was a changed property. 

```
2026-07-17 16:58:06.576506+0200 0x35394    Default     0x0                  110    0    audiomxd: (MediaExperience) [com.apple.coremedia:] -CMVAEndptMgr- cmsmBluetoothListeningModeListener_block_invoke: Listening mode for port=679 changed to ANC
```
`audiomxd` then logs that the listening mode was changed to `ANC`.

```
2026-07-17 16:58:06.591469+0200 0x353d2    Default     0x0                  80     0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AAServicesDaemon] reporting device:
```

```
2026-07-17 16:58:06.591478+0200 0x353d2    Default     0x0                  80     0    audioaccessoryd: (CoreUtils) [com.apple.AudioAccessory:AAServicesDaemon] AADevice id: F4CF7039-6E99-F1D6-DF03-77A730FB317B, nm AirPods Pro, ml AirPodsPro1,1, pid 8206, 
color 255, 
AutoANC: cap -, cfg L, 
, comb LR Batt LR '-90.00%'[ls: 2026-07-17-14:58:04.475, sFL: 0x5 < HeadsetAdv AACPInfo >]Batt L '-92.00%'[ls: 2026-07-17-14:58:04.475, sFL: 0x5 < HeadsetAdv AACPInfo >], Batt R '-90.00%'[ls: 2026-07-17-14:58:04.475, sFL: 0x5 < HeadsetAdv AACPInfo >], 
Bbl: cap -, 
Cam ctl: cap -, 
Chr rem: cap -, DEOC: cap -, OBC: cap +, en +, 
HG: 
HA: cap -, v2 cap -, HP: cap -, PPE: cap -, HT: cap -, 
HR: cap -, 
Lsn: md ANC, cfg 0x6 < ANC Transparency >, 
Pl: md +, pr InEar, sc InEar, 
smRtS 0x4 < Routed >, temp mg -, 
SrMt: cap +, 
Sldt: cap -, 
SS: as Idle, fqBd 2.4, aos None, 
Misc Caps: cas snd -, ear fit -, ff upl -, hide er -, hide off -, ovd str -, pmee -, pt -, wrd lssls -, Misc Info: enh trn V2,lst conn '2026-07-17 14:56:09.000', 

CBDevice F4CF7039-6E99-F1D6-DF03-77A730FB317B, BDA <private>, Nm <private> , Md AirPodsPro1,1, PID 0x200E (AirPodsPro1,1), PrNm AirPods Pro, VID<…>
```
And the `audioaccessoryd` logs the AirPods with the listening mode as transparant: `Lsn: md ANC`.

```
2026-07-17 16:58:10.233915+0200 0x623      Default     0x0                  35     0    SpringBoard: (MediaControls) [com.apple.amp.mediacontrols:Volume] MRUListeningModeController update primary bluetooth to listening mode: AVOutputDeviceBluetoothListeningModeAudioTransparency | AVOutputDeviceBluetoothListeningModeActiveNoiseCancellation | device: <MRAVConcreteOutputDevice:0xb06c29a40 (local) "AirPods Pro" uid="B8:81:FA:11:FF:AB" group_id="12E582F3-C755-452A-93CA-F5E5C802AA4C" bluetooth_id=(null) type=Bluetooth subtype=Headphones clusterType=None group-leader group-contains-leader parent-group-discoverable-leader enc-prog-dl-assets fetch-sender-media-data opt-audio-ui BTHeadphones76,8206 bluetooth-sharing (batt l:0.92|r:0.90) AirPlay infra>
```
After approximately 4 seconds the`SpringBoard` process records an update in the primary bluetooth to listening mode `AVOutputDeviceBluetoothListeningModeAudioTransparency`.

## Conclusion
When the AirPods are used to switch between the listening modes, the first observable event is recorded by `bluetoothd`. This event logs the receipt of a listening mode control command from the paired AirPods. The subsequent records from `audiomxd` and `audioaccessoryd` document the propagation of the new state through the audio subsystem.

When comparing this to the logs found in AUL when the mode is [changed using the iPhone](), the biggest difference is that the `SpringBoard` process, `MediaControls` / `Volume` subsystem logs didn't log setting changed the listening mode, but just logged `update`:

|Action|First process|First evidence|Interpretation|
|---|---|---|---|
|Changed via iPhone|`SpringBoard`|`MRUListeningModeController`|Request originated from Control Center|
|Changed via AirPods|`bluetoothd`|`Received Listen mode (control cmd 0x0D)`|Request originated from the AirPods|