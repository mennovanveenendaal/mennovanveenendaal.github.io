---
title: "Windows Registry 101: A Guide for Forensic Investigations"
layout: post
categories: [Windows, Registry]
image:
  path: /assets/2024/registry/registry.png
---
Every Windows device has a **registry**, but what is its purpose? I delved deep into the Windows Registry to understand its structure and discover why it's crucial for investigators.


## Windows Registry
The Windows Registry [is](https://learn.microsoft.com/en-us/troubleshoot/windows-server/performance/windows-registry-advanced-users) "a central hierarchical database \[...] used to store information that is necessary to configure the system for one or more users, applications, and hardware devices". 

Windows uses the registry to manage (to name a few):
- System settings
- File and folder history
- Which programs open certain file types
- Programs that run at startup
- User accounts and settings
- Time zone information
- Past network connections
- Connected USB devices

### Structure
The information in the Windows Registry is stored in 5 Registry Hives (or root keys). 

1. HKEY_CURRENT_USER
2. HKEY_USERS
3. HKEY_LOCAL_MACHINE
4. HKEY_CLASSES_ROOT
5. HKEY_CURRENT_CONFIG

HKEY stands for **Handle to Registry Key**. These hives (or HKEY) are the place where a specific set of settings is stored.

![root_keys](/assets/2024/registry/root_keys.png)

Each hive contains keys, (like folders)

![keys](/assets/2024/registry/keys.png)

These keys contain subkeys (and some subkeys have subkeys themself) that store settings in **values** (the actual data).

![subkeys](/assets/2024/registry/subkeys.png)

Almost all Windows applications store their configuration data in the registry, though some portable applications may use separate files like `.xml` or `.ini` for this.

### Where are the Hives stored?
Most of the Hives (except HKEY_CURRENT_USER) are located in the `%SystemRoot%\System32\Config` directory.

The user related HKEY_CURRENT_USER is located in the user’s profile folder: `%SystemRoot%\Users\<Username>`

### Root Key description

| Root key                | Description                                                                                                                                   |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| **HKEY_CLASSES_ROOT**   | Manages file type associations, determining which program opens each file type.                                                               |
| **HKEY_CURRENT_USER**   | Stores settings for the currently logged-in user, like display settings, recently opened files, and environment variables (such as the PATH). |
| **HKEY_LOCAL_MACHINE**  | Holds system-wide configurations for hardware and software.                                                                                   |
| **HKEY_USERS**          | Contains information for all user profiles on the system.                                                                                     |
| **HKEY_CURRENT_CONFIG** | Stores the hardware configuration used by the system when starting up.                                                                        |

### Key Details
#### HKEY_CLASSES_ROOT
The keys in this Hives are the different extensions used on the system containing a key with data for the Program that is used to open a file. Combines the user configuration in HKEY_CURRENT_USER\\Software and machine configuration in HKEY_LOCAL_MACHINE\\Software

#### HKEY_USERS
Contains data for each user profile on the system.

#### HKEY_CURRENT_USER
Subkey of HKEY_USERS. **HKEY_USERS** contains the registry settings for **all** users, while **HKEY_CURRENT_USER** is a shortcut to the currently logged-in user’s settings.

Stores the settings in the `NTUSER.DAT` file. This includes environment variables, screen settings, and recently accessed files. File is loaded during user login. 

These settings are loaded when the user logs into the system, and unloaded when the user logs out.

Keys include (o.a.):
- Environment (Path value)
- System (System settings like the homedrive)

#### HKEY_LOCAL_MACHINE
Contains 6 keys:
- BCD
    - Boot configuration (replaces `boot.ini`).
- HARDWARE
    - Stores information about hardware, recreated at every boot.
- SAM
    - Security Account Manager, containing local accounts and groups.
- SECURITY
    - Holds system security policies and permissions.
- SOFTWARE
    - Stores software configurations, including file type associations.
- SYSTEM
    - Contains system-level configurations for Windows.

#### HKEY_CURRENT_CONFIG
Contains current system hardware and software settings.

### Registry Data Types
The data in the Registry is store in **values** (the actual data). These can have multiple types:

| Name                    | Data type                      | Data type                                                                                           |
| ----------------------- | ------------------------------ | --------------------------------------------------------------------------------------------------- |
| Binary Value            | REG_BINARY                     | Binary data.                                                                                        |
| DWORD Value             | REG_DWORD                      | 32-bit integer, commonly used for configuration settings.                                           |
| QWORD Value             | REG_QWORD                      | 64-bit (8 byte) integer. Used when DWORD (32-bit) is insufficient.                                  |
|                         | REG_DWORD_LITTLE_ENDIAN        | 32-bit integer in little-endian. Mostly used.                                                       |
|                         | REG_REG_DWORD_BIG_ENDIAN       | 32-bit integer in big-endian                                                                        |
| Expandable String Value | REG_EXPAND_SZ                  | String. Can contain environment variables (e.g., `%SystemRoot%`). The string expands when called.   |
| Multi-String Value      | REG_MULTI_SZ                   | List of multiple strings, each terminated by a null character. Used for settings with multi-values. |
| String Value            | REG_SZ                         | A fixed-length text string.                                                                         |
| Binary Value            | REG_RESOURCE_LIST              | List  of hardware resources to be used  by the device drivers.                                      |
| Binary Value            | REG_RESOURCE_REQUIREMENTS_LIST | List of possible hardware resources that the system **could** use.                                  |
| Binary Value            | REG_FULL_RESOURCE_DESCRIPTOR   | Detailed list for physical hardware device. Mainly used for Plug-and-play devices.                  |
| None                    | REG_NONE                       | Undefined or raw binary data types.                                                                 |
| Link                    | REG_LINK                       | Symbolic link to other key. Used as shortcut to redirect registry operations to an other key.       |

## Acquiring 
### Using Tools for Acquisition

- [FTK Imager](https://www.exterro.com/digital-forensics-software/ftk-imager): A popular tool for acquiring forensic images of drives, including registry hives, without altering the original data.
- [RegRipper](https://regripper.wordpress.com/): A tool designed for processing registry hives and generating reports, useful for automating analysis.
- **Eric Zimmerman's [tools](https://ericzimmerman.github.io/#!index.md)**: Eric Zimmerman has developed several registry tools, including **Registry Explorer**, which allows investigators to view multiple registry hives simultaneously and create bookmarks for frequently used keys.

### Manual Acquisition
- **regedit**: The built-in Windows Registry Editor (regedit) can be used to view and export registry hives or individual keys manually for analysis. Simply navigate to the key, right-click, and select 'Export' to save it for further investigation.

## Forensic Importance of Registry Keys
The Windows Registry holds a wealth of forensic information, particularly in areas related to user behavior and system configuration. Some of the most important keys for forensic analysis include the following:

### Windows Version and Owner Info
`HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion`: This key contains details about the installed version of Windows, including the registered owner and the version of the OS.

### Software Installations
The `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall` key logs software installations, showing which programs were installed on the system, when, and by which account.

### Time Zone Setting
`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation`: Stores the system's time zone.

### USB devices
`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`: Tracks connected USB storage devices, including device IDs, which can be useful for determining unauthorized data transfers or device connections.

### Network connections and configurations
`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces` stores details of past network connections. This can provide a lot of details when tracing a computer's network activity.

The configuration for a network is found in `SYSTEM\CurrentControlSet\Service\Tcpip\Parameters\Interfaces\{GUID OF NETWORK}\`.

### Autostart programs
The `Run` keys, often in `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run`  show programs that automatically launch on startup. Whit this malware persistence mechanisms can be identified.

### User activity
Several registry keys store critical data on user behavior:

#### User Logons:
Logon events can be found under `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa` .

#### Recent files:
`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` tracks recently accessed files.

#### Open/Save Dialog History:
`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSavePidlMRU` logs files that have been opened or saved via the File Explorer dialog boxes.

#### Browsing:
For **Internet Explorer**, browsing history is stored under `HKEY_CURRENT_USER\Software\Microsoft\Internet Explorer\TypedURLs`.

Chrome user data can be found at `HKEY_CURRENT_USER\Software\Google\Chrome\PreferenceMACs`.

### Shares:
`HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\lanmanserver\Shares` contains information on folders shared over the network.

### Remote Desktop (RDP) Connections:
The `HKEY_CURRENT_USER\Software\Microsoft\Terminal Server Client\Default` key holds information regarding recent RDP connections, including the IP addresses of remote systems accessed from the device.

### ShellBags:
`HKEY_USERS\<SID>\Software\Microsoft\Windows\Shell\BagMRU`: Records folder views and settings, enabling the reconstruction of user activity and accessed folders, even if the folder has been deleted.

## Conclusion
The Windows Registry is an invaluable source of forensic evidence. By analyzing registry data, investigators can uncover user behavior, track connected devices and understand system configurations. Mastery of registry analysis is essential for digital forensics, as it provides a clear timeline of activity and crucial system events.