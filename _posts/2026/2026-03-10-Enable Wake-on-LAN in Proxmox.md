---
title: "Enable Wake-on-LAN in Proxmox"
layout: post
categories: [Homelab, Proxmox]
description: Enabling WOL on a Proxmox node.
image:
  path: /assets/2026/proxmox/wol.png
  alt: Wake-on-LAN
---

Sometimes I need to start one of my Proxmox nodes while I am away from home. To make this possible, I enabled Wake on LAN on all nodes. Many articles describe this process. This post summarizes the steps that worked for me.

## ethtool
To configure Wake on LAN, first install [ethtool](https://man7.org/linux/man-pages/man8/ethtool.8.html) 
```shell
apt install ethtool
```

To check whether Wake on LAN is already enabled, determine the network interface name. 
```shell
ip a
```

Then inspect the network interface with:
```shell
ethtool eth0
```

Look for the lines `Supports Wake-on:` and `Wake-on:` and check the [status](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/6/html/deployment_guide/s1-ethtool). The  `Supports Wake-on:` line displays all supported modes, while  `Wake-on:` displays the currently active mode.
```shell
- `p` — Wake on PHY activity.
- `u` — Wake on unicast messages.
- `m` — Wake on multicast messages.
- `b` — Wake on broadcast messages.
- `g` — Wake-on-Lan; wake on receipt of a "magic packet".
- `s` — Enable security function using password for Wake-on-Lan.
- `d` — Disable Wake-on-Lan and clear all settings.
```

## Enabling Wake on LAN 
To enable Wake on LAN using a magic packet:
```shell
ethtool -s eth0 wol g
```

Command options:
```shell
-s Modifies network device settings.  
wol Configures Wake on LAN options.  
g Enables wake up through a magic packet.
```

### Persistent configuration
The command above enables Wake on LAN immediately, but the setting may not survive a reboot. To make the configuration persistent, add the following line in `/etc/network/interfaces` to the active interface configuration.

```shell
auto eth0
iface eth0 inet manual
	post-up /usr/sbin/ethtool -s eth0 wol g
```

Save the file and reload the network configuration or reboot the node.

## Proxmox GUI
After reloading the network configuration or rebooting the node, open the Proxmox web interface.  
  
Navigate to System, Network, select the network interface, click Edit, verify that Autostart is enabled, and save the configuration. Although not required for Wake on LAN itself, enabling Autostart ensures the network interface is automatically activated after the system boots.

## Motherboard settings
Note that not all hardware supports Wake on LAN.
If the hardware supports Wake on LAN, it can be enabled in the BIOS or UEFI settings of the system.

Restart the node and enter the firmware configuration menu. The required key differs by manufacturer.

The Wake on LAN option is commonly located under Advanced (Settings), Power Management, or a similar menu. Enable Wake on LAN for the correct network adapter and save the configuration.
