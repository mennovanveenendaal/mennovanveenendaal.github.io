---
title: "Building a "
layout: post
categories: [Proxmox, Building]
image:
  path: /assets/2025/phones/phone.png
  alt: T65 Rembrandt
description: 
---

Years ago I received an old T65 Rembrandt rotary phone. I liked its aesthetic and wanted to give the phone a practical function. I connected an Arduino Nano to play music through the handset and to detect pulses from the rotary dial. Based on the number of pulses I toggled a light switch in Domoticz. The connection was made using [MySensors](https://www.mysensors.org/) and an NRF24L01 antenna. When I switched from Domoticz to Home Assistant the system disappeared into the background.

## Resurrecting the project
The phone remained on my desk along with the idea of connecting it to Home Assistant. In Home Assistant 2023.5 voice support was introduced. This made it possible to connect a [landline phone](https://www.home-assistant.io/voice_control/worlds-most-private-voice-assistant/) as a voice interface.

Using a Grandstream HT801 I connected the rotary phone. After lifting the handset I could speak to the server. This worked surprisingly well.

## Calls
To expand the setup I wanted the phones to make internal calls. I found a YouTube video by [Network Chuck](https://www.youtube.com/watch?v=fdM1V98iIQI) where he used 3CX to enable rotary phone calls.

The 3CX system can run in the cloud or locally. I wanted it to run locally.

### Creating the system
In Proxmox I created a VM for 3CX. The VM had a 10 GB boot disk, 1 CPU and 4 GB memory.

Following the video I configured both 3CX and the Grandstream. I removed the 3CX server path in the Grandstream (Maintenance > Upgrade > Config File > Config Server Path) to prevent 3CX from overwriting my custom configuration.

## Pulse dial
To make calls I enabled the pulse dial function in the HT801 (Port Settings > FXS port > Audio Settings > Enable Pulse Dialing). With this setting the Grandstream detected the pulses from the rotary dial.

## Ring
The phones worked and the logs showed that calls were being placed. However the phones did not ring when receiving a call. After a long search I discovered that I needed to modify the [wiring]( https://www.matilo.eu/technish/oude-telefoon-aansluiten/) to activate the internal bells. The wire for the extra bell (yellow) needed to be connected to the blue wire. I made this connection in the socket to avoid damaging the phones or their plugs.

## Final call
With this final step the internal phone system worked completely. I can now make and receive calls between the phones and Home Assistant, giving a decorative object a new functional purpose.
