---
title: "Data exfiltration using GitHub"
layout: post
categories: [Windows, Registry]
image:
  path: /assets/2024/dataexfil/github.png
  alt: GitHub
---

Windows 10 home 22H2 clean install
git version 2.47.0.windows.1
FTK Imager 4.7.1.2

```bash
git config --global user.email "<my@email.biz>"
git config --global user.name "<user.name>"
echo "# exfiltration_test" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/mennovanveenendaal/exfiltration_test.git
git push -u origin **main**
```

Download chirpy repository in .zip.

Small cleanup, 

- 48 folders
- 198 files
- size 493986 bytes
- size on disk 770048 bytes

Difference size https://superuser.com/a/66826

```shell
git add --all
git commit -m "data exfiltration"
git push -u origin main
```

# FTK
Add C:\ drive as Logical drive
export disk image Raw

Transfer to investigation VM.

Copy evidence.
Import this copy in FTK as evidence:
![FTK open](/assets/2024/dataexfil/FTK_open.png)

- Was GitHub installed on this device?
- Was Github used on this device?
- 

![Registry Keys in Registry Explorer](/assets/2024/dataexfil/keys.png)


**Device information**
`SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName` DESKTOP-2D85SP7

`SYSTEM\CurrentControlSet\Control\TimeZoneInformation` 
- TimeZoneKeyName: Pacific Standard Time
- ActiveTimeBias: 420
- DaylightName -211 == [Pacific Standard Time](https://www.nirsoft.net/dll_information/windows8/tzres_dll.html)

`SOFTWARE\Microsoft\Windows NT\CurrentVersion` 
- ProductName: Windows 10 Home
- DisplayVersion: 22H2
- InstallDate: 1722450503 [\==](https://www.unixtimestamp.com/) Wed Jul 31 2024 18:28:23 GMT+0000
- SystemRoot: C:\Windows



**Locate Git Configuration and History Files**


**SSH Keys for GitHub**
`HKEY_CURRENT_USER\Software\OpenSSH\Agent`

**Installed Git Software**
`HKEY_LOCAL_MACHINE\Software\GitForWindows`
`HKEY_CURRENT_USER\Software\GitForWindows`

**Environment Variables (GitHub in Command Line)**
`HKEY_CURRENT_USER\Environment`
`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment`
**What to look for**: Look for environment variables like `GIT_SSH` or `HTTPS_PROXY`, which might indicate how GitHub connections were established. This can help you track specific tools or commands used to access GitHub repositories.

**User Preferences in Git GUI Clients**
`HKEY_CURRENT_USER\Software\GitHub\GitHubDesktop`

**HTTP Proxies or Certificates**
`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings`

**Task Scheduler (For Automated Uploads)**
`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree`

**Network Activity Logs**
Out of scope

