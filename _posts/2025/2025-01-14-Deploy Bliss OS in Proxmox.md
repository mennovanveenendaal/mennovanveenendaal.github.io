---
title: "Deploy Bliss OS in Proxmox"
layout: post
categories: [Proxmox, BlissOS]
image:
  path: /assets/2025/proxmox/blissos.png
  alt: Bliss OS
---
To analyze Android app behavior and run mobile applications without needing an extra physical device, I set up [BlissOS](https://blissos.org/) in a virtual machine. This approach allows for easy resets to a clean state and isolates the environment within a dedicated VLAN to protect the rest of my network.

BlissOS, an open-source Android-based operating system, is well-suited for this use case and can be installed on Proxmox. I referred to [Novaspirit Tech](https://www.youtube.com/watch?v=LEyElt_yP50) for inspiration and followed these steps:

## ISO Preparation

1. Download the [BlissOS 16 ISO](https://blissos.org/index.html#download) and import it into Proxmox.
2. Install the necessary libraries on the Proxmox node:
   `apt install libgl1 libegl1`

## VM setup
### General
![General](/assets/2025/proxmox/general.png)
_Fig.1 General_

Assign an ID and name the VM.

### OS
![OS](/assets/2025/proxmox/os.png)
_Fig.2 Operation System_

Select the BlissOS ISO, set the Type to **Linux**, and choose **6.x - 2.6 Kernel** as the version.

### System
![System](/assets/2025/proxmox/system.png)
_Fig.3 System_

The settings that worked for me where 
- Graphics Card: VirtIO-GPU
- Machine: q35
- BIOS: OVMF (UEFI)
- EFI disk

### Disks
![Disks](/assets/2025/proxmox/disks.png)
_Fig.4 Disks_

### CPU
![CPU](/assets/2025/proxmox/cpu.png)
_Fig.5 CPU_

### Memory
![Memory](/assets/2025/proxmox/memory.png)
_Fig.6 Memory_

### Network
![Network](/assets/2025/proxmox/network.png)
_Fig.7 Network_

## Installing BlissOS
Start the VM and select the installation option.

![Installation](/assets/2025/proxmox/installation.png)
_Fig.8 Installation_

To start, the partitions needed to be created:
![Create](/assets/2025/proxmox/create.png)
_Fig.9 Create partitions_

Use `cfdisk` and select `gpt` as the label type.

![cfdisk](/assets/2025/proxmox/cfdisk.png)
_Fig.10 Select cfdisk_

Then select new.
![new](/assets/2025/proxmox/new.png)
_Fig.11 Select new_

Create two partitions:
- A 1 GiB EFI partition with type **EFI**.
- A second partition using the remaining space (e.g., 127 GiB).
- 
![1GiB](/assets/2025/proxmox/1g.png)
_Fig.12 1 GiB_

Select `free space`, and `new` again and create the second partition, with a partition size of 127G.

After that `write` and `quit`:
![write](/assets/2025/proxmox/write.png)
_Fig.13 Write_

## Format partitions
Format partition 1 (sda1) as FAT32 with the name `esp`.

Select the partition
![format](/assets/2025/proxmox/format.png)
_Fig.14 Format the first partition_

Selected `FAT32`, leave the default name to `esp` and select `yes` to format the partition.

Format partition 2 (sda2) as ext4 with the name `BlissOS`.

Enable OTA updates and install **Grub2** as the bootloader.

![ota](/assets/2025/proxmox/ota.png)
_Fig.15 OTA_

![Grub](/assets/2025/proxmox/grub.png)
_Fig.16 Grub2 Bootloader_

## Reboot and Configure BlissOS
After installation, reboot the VM. Once BlissOS starts, follow the setup steps provided by the OS.

![blissos](/assets/2025/proxmox/blissos.png)
_Fig.18 BlissOS_

## Play Store
To enable the Play Store, activate it within BlissOS. For verification, I used a prepaid SIM card to obtain the activation code.

This setup within Proxmox provides a versatile, isolated environment for Android app testing and exploration. Check out the [BlissOS website](https://blissos.org/) for more details!

## Taskbar
While working in BlissOS via the Console, I found that some applications rotate the view, leaving them unable to control. To fix this I adjusted the Taskbar settings by going into Settings -> Apps -> Default apps -> Home app. There I selected Taskbar  for Bliss OS. 

![taskbar](/assets/2025/proxmox/taskbar.png)
_Fig.19 Select Taskbar for Bliss OS_