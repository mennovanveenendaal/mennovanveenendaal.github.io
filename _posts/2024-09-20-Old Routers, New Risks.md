---
title: Old Routers, New Risks
layout: post
categories: Hardware hacking, investigating 
image:
  path: /assets/old_router/router.png
  alt: Router
---
[Recycling](https://www.epa.gov/recycle) is becoming more common, and consumers are increasingly encouraged to [resell](https://www.watismijnapparaatwaard.nl/verkopen) their old devices rather than throw them away. While selling old hardware might seem harmless, it can also pose unexpected risks. Unwiped routers, for example, can still contain sensitive information. In this article, I investigate a second-hand router to see if the data left on it could be useful to an investigator, or an attacker.


# Hands-on with the TP-Link WR841n router
In courses like the _Beginner’s Guide to IoT and Hardware Hacking_ by [TCM Security](https://academy.tcm-sec.com/p/beginner-s-guide-to-iot-and-hardware-hacking) practical examples on real devices are used to teach the fundamentals of IoT hacking. One recommended device for these tasks is the TP-Link WR841n router, ideal for exercises like sniffing communications, taking measurements, and extracting firmware.

To gain hands-on experience, I purchased a second-hand TP-Link WR841n from Marktplaats.nl (a Dutch equivalent of eBay). When I arrived to pick it up, there was a brief delay at the door, almost as if the seller had forgotten about the sale and quickly unplugged the router from their network.

# Initial Investigating
Although the course didn’t require it, I like to investigate devices before physically opening them. This involves trying to gather as much information as possible without altering or tampering with the device—like checking if a door is unlocked before breaking it open.

Since this router had already been powered down, I assumed any logs were wiped. Feeling confident, I decided to break the usual rule of “if it’s off, leave it off” since it was just a practice device with no active internet connection (no LAN cable or mobile network support).

# Accessing the Router's GUI
Once powered on, the router broadcasted a visible network:

![SSID](/assets/old_router/SSID.png)
_Fig.1 SSID_

I tried connecting using the password on the back of the device, but it was incorrect. Knowing that LAN ports on routers like this are usually [insecure](https://www.kaspersky.com/blog/dangerous-ethernet-ports/31289/), I connected a LAN cable directly to the router. My laptop received an IP address of 192.168.0.100, and the router was assigned to 192.168.0.1.

I accessed the router’s login page at  http://192.168.0.1.

![login](/assets/old_router/login.png)
_Fig.2 Login page_

Before searching online for default credentials, I tried “admin/admin, and it worked!

![gui](/assets/old_router/gui.png)
_Fig.3 Inside the GUI_

## Further Investigation
Although all logs had been cleared, there still can be useful information in the router’s GUI. For instance, under "Advanced Routing," you could find details about the previous network the device was part of. Additionally, static IP addresses assigned to specific MAC addresses could reveal information about devices previously connected to this router, potentially identifying devices (and with that also potentially people) once within the router’s range. However, further investigation would be needed to gain more insights and draw conclusions.

For an attacker, this router could hold valuable information. IP settings like ranges and DHCP pools, firewall rules can all be useful for mapping out a network. This information could then be exploited in an attack on a new network.

And the GUI could also reveal the Wi-Fi password in plain text.

## Finding the Wi-Fi Password
The Wi-Fi password had been changed. Under “Wireless Security” in the Wireless settings, I was able to find the current password for the wireless network:

![password](/assets/old_router/password.png)
_Fig.4 Password_

# Investigating the Router’s Location
With the rise of IoT devices, households are connecting more and more appliances to their wireless networks. Since updating the Wi-Fi settings in all devices can be a hassle, many people don't bother changing their SSID or there Wi-Fi passwords when swapping out routers. Instead, in there new router, they set these settings to match the old ones, so all devices can easily connect again.

## Using OSINT to Identify the Router’s Location
When purchasing a second-hand router at a private sellers home, you have a strong indication where it was used. But if you acquire one from a thrift shop or online marketplace, you can use OSINT (Open Source Intelligence) techniques to locate where the router was used.

For this device I used [Wigle](https://wigle.net/) a website that tracks the location of wireless networks. By entering the router’s MAC address, I pinpointed the general location where the router had been used.

![Wigle](/assets/old_router/wigle.png)
_Fig.5 Wigle GUI_

With this information, an attacker could potentially locate the network and attempt to access it. Once connected, they could explore or attack other devices on the network.

# The Importance of Resetting Your Router
If you're selling or giving away your router, the new owner might not have malicious intentions, but it's better to be safe than sorry. A simple reset prevents them from accessing your old network details, saved passwords, or any other personal data that might still be stored.

## How to Properly Reset a Router
![reset](/assets/old_router/reset.jpg)
_Fig.6 Reset button_

To reset a router, most models (like the TP-Link WR841n) have a small reset button. This an be pressed using a paperclip or pin and needs to be hold for about 10 seconds. Once done, the router will reboot and revert to its original factory settings, including the default login credentials and factory-assigned wireless SSID and password.