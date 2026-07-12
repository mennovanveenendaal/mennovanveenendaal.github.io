---
title: "AirPods and the Apple Unified Log. Part 10. Conclusion"
layout: post
categories: [iOS, AirPods]
description: Investigating the Apple Unified Logging to find artefacts of connected Airpods. Part 10
image:
  path: /assets/2026/airpods/airpods_10.png
  alt: FindMy
---
This post is the tenth article in a serie that examines which AirPods related artefacts are recorded in the Apple Unified Log, concluding the serie.

## Closing the case
The Apple Unified Log contains sufficient information to reconstruct nearly the complete interaction between an iPhone and a paired set of AirPods. Events such as opening the charging case, removing individual AirPods, detecting whether each AirPod is in the ear or in the case, initiating media playback, pausing playback, and returning the AirPods to the Charging Case can all be identified.

Several subsystems contribute to this information, including `audioaccessoryd`, `bluetoothd`, `sharingd`, and `findmydeviced`. Correlating these subsystems provides a detailed timeline of user interaction.

Although the volume of logging is considerable, it also presents a challenge. Relevant events are distributed across multiple subsystems and categories, making correlation essential. These artefacts can provide valuable evidence of AirPods usage and user interaction with an iPhone.

## What could not be found in the Apple Unified Logs
This logs did not record which AirPod generated the Play and Pause commands. Closing the lid of the Charging Case when both AirPods where out of the case was also not found in the current logs.

The timestamps in the logs reflect when the iOS operating system processed and logged the events. This timestamp may differ (slightly) from the exact moment the user performed the action.

## Overview of the articles
- [Part 1. Introduction and methodology](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-1.-Introduction-and-methodology/)
- [Part 2. Opening the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-2.-Opening-the-Charging-Case/)
- [Part 3. Detecting AirPods removed and inserted into the ear](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-3.-Detecting-AirPods-removed-and-inserted-into-the-ear/)
- [Part 4. Closing the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-4.-Closing-the-Charging-Case/)
- [Part 5. Media play event](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-5.-Media-play-event/)
- [Part 6. Media pause events](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-6.-Media-pausing-event/)
- [Part 7. Detecting AirPods removed from the ears](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-7.-Detecting-AirPods-removed-from-the-ears/)
- [Part 8. Detecting AirPods returned to the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-8.-Detecting-AirPods-returned-to-the-Charging-Case/)
- [Part 9. Closing the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-9.-Closing-the-Charging-Case/)
- Part 10. Conclusion