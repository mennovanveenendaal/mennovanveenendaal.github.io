---
title: "AirPods and the Apple Unified Log. Part 4. Closing the Charging Case"
layout: post
categories: [iOS, AirPods]
description: Investigating the Apple Unified Logging to find artefacts of connected Airpods. Part 4
image:
  path: /assets/2026/airpods/airpods_4.png
  alt: FindMy
---

This post is the fourth in a serie that examines which AirPods related artefacts are recorded in the Apple Unified Log. This post focuses on closing the Charging Case.

## Closed Charging Case
Opening generates numerous events. The logs recorded the moment a connection was made and when the Airpods where removed.
However, after removing the Airpods and closing the Charging Case, no log entry was found that records the moment the Charging Case was closed.

This reflects the use of the Airpods. When the Airports are in the Charging Case the iPhone also shows the case itself:

![With Case](/assets/2026/airpods/AirPods_Case_Power.png)
_Fig.1 AirPods in Charging Case_

When the AirPods are outside of the case and the lid of the case is closed, the case is no longer showed on the iPhone:

![Just Airpods](/assets/2026/airpods/AirPods_Power.png)
_Fig.2 AirPods outside Charging Case_

## Indirect logging
When the Charging Case is open it is recorded as Power Source. When the AirPods are outside the case and the Charging Case is closed, only the AirPods are recorded.

Between timestamps `16:19:03.424` and `16:19:16.653` no  logging was found recording the battery status of the Charging Case. 

## Overview of the articles
- [Part 1. Introduction and methodology](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-1.-Introduction-and-methodology/)
- [Part 2. Opening the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-2.-Opening-the-Charging-Case/)
- [Part 3. Detecting AirPods removed and inserted into the ear](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-3.-Detecting-AirPods-removed-and-inserted-into-the-ear/)
- Part 4. Closing the Charging Case
- [Part 5. Media play event](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-5.-Media-play-event/)
- [Part 6. Media pause events](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-6.-Media-pausing-event/)
- [Part 7. Detecting AirPods removed from the ears](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-7.-Detecting-AirPods-removed-from-the-ears/)
- [Part 8. Detecting AirPods returned to the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-8.-Detecting-AirPods-returned-to-the-Charging-Case/)
- [Part 9. Closing the Charging Case](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-9.-Closing-the-Charging-Case/)
- [Part 10. Conclusion](https://www.mennovanveenendaal.com/posts/AirPods-and-the-Apple-Unified-Log.-Part-10.-Conclusion/)