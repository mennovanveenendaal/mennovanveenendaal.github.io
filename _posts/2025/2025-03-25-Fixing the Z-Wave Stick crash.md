s---
title: "Fixing the Z-Wave crash"
layout: post
categories: [Home Automation, Z-Wave]
image:
  path: /assets/2025/zwave/Z-Stick.jpg
  alt: Z-Wave Stick
---

In our household, I automate as much as possible to make things more efficiency. And because I enjoy the challenge.

To interact with the different sensors and lights I use both Zigbee and Z-Wave. For the latter I use a Aeotec Z-Wave Plus Z-Stick Gen5 to connect the devices to my Home Assistant server.

## Crash
This setup worked fine until I noticed a recurring problem. Every time Home Assistant, the Z-Wave Add-On, or the system rebooted, the Z-Wave Add-On failed to detect the Z-Stick. As a result, all my Z-Wave devices became unresponsive.

After searching online I tried various fixes, like updating the firmware on the stick, resetting the Z-Stick and re-add all the devices and a clean install of Home Assistant. Non of them worked, so I left the Z-Stick out of the system.

## Reconnecting
When I plugged the Z-Stick back into the system after a reboot after an update, I noticed that the Z-Stick was recognized by the Z-Wave Add-On, and that Home Assistant could interact with the sensors again. When I tried to recreate this, unplugged the Z-Stick and rebooted the system and re-plugged the Z-Stick again, I found that the Z-Stick was indeed recognized again.

## Solution
One day, after a system update, I plugged the Z-Stick back in after a reboot and it worked! Home Assistant recognized it, and all sensors came back online.

Testing further, I found that If the Z-Stick was unplugged before rebooting and plugged back in afterward, it was detected normally. If left plugged in during reboot, it wasn’t recognized by Home Assistant.

## Automatings
Unplugging and replugging the Z-Stick manually was impractical, especially if I needed to reboot remotely. I needed an automated solution.

I found the [Sonoff Micro Zigbee USB smart adaptor](https://sonoff.tech/product/diy-smart-switches/micro/), a USB switch controlled via Zigbee (since using Z-Wave to fix a Z-Wave issue wouldn’t work).

After pairing the USB adaptor with Home Assistant (Zigbee controller in inclusion modus, and powering the USB adaptor on) it was recognized instantly. 

## Code
I wrote two simple automations; one that would power the Z-Wave stick off when Home Assistent shuts down and one that would power the Z-Wave stick back up when Home Assistent was booted again.

### Automation Code
#### Power Off (Before Shutdown):

```yaml
- id: "<INSERT ID>"
  alias: "<INSERT ALIAS>"
  trigger:
    - platform: homeassistant
      event: shutdown
  action:
    - service: switch.turn_off
      target:
        entity_id: switch.zwave_switch
```

#### Power On (After Startup):
```yaml
- id: "<INSERT ID>"
  alias: "<INSERT ALIAS>"
  trigger:
    - platform: homeassistant
      event: start
  action:
    - delay:
        seconds: 60
    - service: switch.turn_on
      target:
        entity_id: switch.zwave_switch
```

## Conclusion
While I never found the root cause of the issue, this workaround solved the problem. Now, I can reboot or update Home Assistant without losing Z-Wave connectivity or manually unplugging the Z-Stick.


