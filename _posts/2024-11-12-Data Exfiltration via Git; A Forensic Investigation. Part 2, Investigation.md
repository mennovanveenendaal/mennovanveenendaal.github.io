---
title: "Data Exfiltration via Git: A Forensic Investigation. Part 2, Investigation"
layout: post
categories: [Windows, Registry]
image:
  path: /assets/2024/dataexfil/data_exfiltration.png
  alt: Data exfiltration
---
In [part 1](https://www.mennovanveenendaal.com/posts/Data-Exfiltration-via-Git;-A-Forensic-Investigation.-Part-1,-Exfiltration/) of this blog data was exfiltrated from a workstation to GitHub using Git.
In part 2, we’ll dive into investigating the data on a Windows 10 workstation to determine whether any sensitive data was exfiltrated to GitHub using Git. The investigation focuses on analyzing traces left on the device, excluding network activity logs.

## Investigation Scope
For this investigation I will focus on the data on the Windows 10 device itself, excluding other sources like network activity logs.

The goal is to determine whether sensitive data had been sent from the device to GitHub using Git. The investigation revolves around key questions:
- [[#Was Git installed?|Was Git installed on this device?]]
- [[#Was Git executed?|Was Git executed on this device?]]
- [[#Was data sent to GitHub?|Are there traces of data being sent to GitHub?]]
- [[#What data was sent?|If so, what data was sent,]] [[#To which repository was data sent?|and to which repository?]]

I will use these questions as structure in this post.

## Acquiring the Disk Image
To begin the investigation, I used FTK Imager to capture the virtual machine’s disk image,  and transferred the image to a separate Windows 10 virtual machine for analysis. There I copied the image to ensure that the original data remained unaltered, preserving its forensic integrity.

On this investigation VM, I loaded the copy in FTK:
![FTK open](/assets/2024/dataexfil/FTK_open.png)
_Fig.1 FTK_

To have a starting point I exported the [registry files](https://www.mennovanveenendaal.com/posts/Windows-Registry-101,-A-Guide-for-Forensic-Investigations/) from this image. From `%SystemRoot%\system32\config\` the `DEFALT`, `SAM`, `SECURITY`, `SYSTEM` and `SOFTWARE` registry files. And from the `<user folder>\` the `NTUSER.DAT` registry file. 

After exporting I uploaded the hives in Registry Explorer.

![Registry Keys in Registry Explorer](/assets/2024/dataexfil/keys.png)
_Fig.2 Registry Keys_

## General System Information
To start the investigation I first investigated the device itself to determine the type, OS, general configurations and user accounts on the device.

### Device information
From the system’s registry, I retrieved the following information:
- **Device name:** "DESKTOP-2D85SP7"
- **Operating system:** Windows 10 Home, version 22H2
- **Timezone:** Pacific Standard Time (UTC-7)
- **User accounts:** Default accounts and a user account named "menno."

### Device name
In `SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName` the device name was set to "DESKTOP-2D85SP7".

### Operating system
In the subkey `SOFTWARE\Microsoft\Windows NT\CurrentVersion` the following values where stored:
- ProductName: Windows 10 Home
- DisplayVersion: 22H2
- InstallDate: 1722450503 [\==](https://www.unixtimestamp.com/) Wed Jul 31 2024 18:28:23 GMT+0000
- SystemRoot: C:\Windows
- RegisteredOwner: Menno

### Timezone
In `SYSTEM\CurrentControlSet\Control\TimeZoneInformation` I found information regarding the timezone setting of this device. This could later help to translate possible timestamps:
- The `TimeZoneKeyName` subkey was set to `Pacific Standard Time`
- The ActiveTimeBias (the number of minutes offset from UTC) was 420. 420 divided by 60 gives an offset of 7, or **UTC-7**.
- The value of DaylightName was -211, translating to [Pacific Standard Time](https://www.nirsoft.net/dll_information/windows8/tzres_dll.html).

### Users accounts
In the `SAM\Users\Names` (Security Account Manager) registry five accounts where present, four [default accounts](https://learn.microsoft.com/en-us/windows/security/identity-protection/access-control/local-accounts) and one user account:
- Administrator
- DefaultAccount
- Guest
- menno
- WDAGUtilityAccount 

Looking at the user account that was laslty used to login:
`SOFTWARE\Microsoft\CurrentVersion\Authentication\LogonUI`
- LastLoggedOnUser: menno

## Was Git installed?
To determine if GitHub was installed on this device I searched for traces related to this program. First I checked the `SOFTWARE` hive and found that GitHub was, indeed, installed.
`Software\GitForWindows` 
- CurrentVersion: 2.47.0
- InstallPath: C:\Program Files\Git

To find how GitHub was used I searched the `NTUSER.DAT` registry. In the console subkey I found the values, indicating that Git could be used via the command interpreter.
`NTUSER.DAT\Console\Git Bash`
`NTUSER.DAT\Console\Git CMD`

In the SYSTEM hive I looked for the values stored in PATH, and also found Git cmd: 
`SYSTEM\CurrentControlSet\Control\Session Manager\Environment`
- path: `%SystemRoot%\system32;%SystemRoot%;%SystemRoot%\System32\Wbem;%SYSTEMROOT%\System32\WindowsPowerShell\v1.0\;%SYSTEMROOT%\System32\OpenSSH\;C:\Program Files\Git\cmd`

However, no SSH keys were present at the time of the investigation for this user account:
`NTUSER.DAT\Software\OpenSSH\Agent` was empty.

### Browser history
Git for Windows can be [downloaded](https://git-scm.com/downloads/win) via the browser, and the browser history can be viewed. 

For Chrome this file was located in `C:\Users\menno\AppData\Local\Google\Chrome\User Data\Default\`. After locating the file in FTK I exported it, and viewed in **DB Browser**. In the database I filtered for git, and found a Google search `git windows`, an URL `https://git-scm.com/downloads/win` and a reference to a file `///C:/Program%20Files/Git/ReleaseNotes.html`.

There where no other browsers on this system.

### Conclusion
Based on this we can conclude that git was downloaded by user account `menno` via Chrome, and was installed on this system.

## Was Git executed?
For this second question I looked for the Git Configuration and history Files in FTK. 
The installation folder `C:\Program Files\Git` and installer `C:\Users\menno\Downloads\Git.2.47.0-64-bit.exe` where in the filesystem. 

The configuration file for git is placed in the user folder. This file could contains the configurations for the user, like username and remote repositories. While the file was present on this system, it only contained user settings.

`C:\Users\menno\.gitconfig`
```shell
[user]
	name = <username>
	email = <email@address.com>
```

The sub-key `C:\Users\menno\AppData\Local\Temp` didn't provide relevant data. 

### Windows Firewall
The Windows Firewall could hold information regarding network traffic and connections, if network logging is enabled. However `C:\Windows\System32\LogFiles\Firewall` was empty, meaning network logging was disabled.

### Application Event Logs
Switching to the data in the Application Event Logs. These are stored in `C:\Windows\System32\winevt\Logs`. Via FTK I exported the logs, and processed theme with Eric Zimmermans `EvtxeCmd`: `EvtxECmd -d "C:\Users\menno\Documents\evidence\exports\Logs" --csv "C:\Users\menno\Documents\evidence\exports\logs-export"`. In the csv file there where multiple entries for git.exe.

### Traces of Program execution
The **Amcache**  stores information about program executions on an Windows system, and is located in `C:\Windows\AppCompat\Programs\Amcache.hve`.

After exporting the Amcache file, I processed it with Eric Zimmermans `AmcacheParser`:
`AmcacheParser -f "C:\Users\menno\Documents\evidence\exports\Amcache.hve" -i --csv "C:\Users\menno\Documents\evidence\exports\Amcache"`
In this csv where also multiple entries who showed that the application Git.exe was executed on this system.

### Conclusion
Based on the traces in the **Application Event Logs** and the **Amcache** we can conclude that git was executed on this system using the `menno` user account.

## Was data sent to GitHub?
After reviewing the **Amcache** data in the csv file that AmcacheParser produced I found entries for the application `git-http-push.exe`.  This application is used bij GIT to [push data](https://git-scm.com/docs/git-http-push) to a remote repository.

`Git,000037ead7a90e1a0d39d6b8c02d404364ca0000ffff,2024-10-11 10:44:46,92b181ef6f8562f30995989dd8a9f1ca2b6cd05e,False,c:\program files\git\mingw64\libexec\git-core\git-http-push.exe,git-http-push.exe,.exe,2024-10-08 08:53:55,git,2492296,2.47.0.windows.1,2.47.0.windows.1,git-http-push.ex|68d3cf7d040e7329,pe64_amd64,False,2.47.0.1,2.47.0.1,510571216,1033,`

### Conclusion
The application git-http-push.exe was executed on this system, indicating that most likely data was sent to a remote repository, or an attempted was made to do so.  

## To which repository was data sent?
Besides the Amcache, the `Prefetch files` could also contain information about program execution on a Windows system. These files are located in `C:\Windows\Prefetch`. After locating these in FTK, and exporting the folder I processed the files using Eric Zimmermans PECmd: `PECmd -d "C:\Users\menno\Documents\evidence\exports\Prefetch" --html "C:\Users\menno\Documents\evidence\exports\pecmd"` 

As the output I selected `html`, given the large amount of data this gave a cleaner overview than an csv file, as displayed in figure 3.

![PECmd](/assets/2024/dataexfil/PECmd_output.png)
_Fig.3 Output of PECmd_

### Repository
When searching this .html file for entries related to `.git` I found multiple enties for GIT.exe and related Git [commands](https://git-scm.com/docs/git#_main_porcelain_commands). These where used from within one folder: `\USERS\MENNO\DOCUMENTS\SECRET_CODE\`.

In FTK I located this folder:

![Secret Code](/assets/2024/dataexfil/secret_code.png)
_Fig.4 Folder in FTK_

At the time of acquiring the disk of this device, this folder contained a `.git folder` and the source code for [Jekyll Chirpy Theme](https://www.mennovanveenendaal.com/posts/Setting-up-the-Chirpy-theme/).

Using this [blog](https://jmeridth.com/posts/git-for-windows-developers-git-series-part-3/) I investigated the `.git` folder. In FTK I could access the folder, and view the files inside.

The file `.git/logs/HEAD` is used to track recent commits and pushes by git. In this there was an entry stating `commit: data exfiltration`.

The `.git/config` file contains information about the remote repository. After opening 
```shell
[core]
	repositoryformatversion = 0
	filemode = false
	bare = false
	logallrefupdates = true
	symlinks = false
	ignorecase = true
[remote "origin"]
	url = https://github.com/mennovanveenendaal/exfiltration_test.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "main"]
	remote = origin
	merge = refs/heads/main
```

### Conclusion
Based on the `.git/config` file, the configured remote repository for the repository in the folder `SECRET_CODE` was `https://github.com/mennovanveenendaal/exfiltration_test.git`. 

## What data was sent?
At the time of acquiring the disk image the `SECRET_CODE` folder contained the source code for the Chirpy Theme. However, this could have been changed after data was uploaded.

To investigate the git repository I exported the `.git` folder and installed Git on my investigation VM.

The file `.git/index` [is](https://shafiul.github.io//gitbook/7_the_git_index.html) "as a sort of temporary staging area, which is filled with a tree which you are in the process of working on.". With the command `git ls-files --stage` the filename are shown, not the actual data inside the files.

Using the help command `git -h` I found that git stores a log off all commits:
![Git Log](/assets/2024/dataexfil/git_log.png)
_Fig.5 Git log_

Adding `shortstat` gives the stats off all commits.
```shell
> git log --shortstat
commit 2aa4ec40961402ccde72413112d03de52c098de6 (HEAD -> main, origin/main)
Author: mennovanveenendaal <EMAIL>
Date:   Fri DATE

    data exfiltration

 169 files changed, 11648 insertions(+)
```
As shown here, 169 files changed. The repository contained 198 files, this difference could be due to .gitignore file excluding files from uploading.

To see the data sent during the commits I used the `git show` [command](https://git-scm.com/docs/user-manual.html#understanding-commits, and stored the results in a text file:
```shell
> git show > ../git_show.txt 2>&1
> cd ..
> more git_show.txt
commit 2aa4ec40961402ccde72413112d03de52c098de6
Author: mennovanveenendaal <EMAIL>
Date:   Fri DATE

    data exfiltration

diff --git a/.editorconfig b/.editorconfig
new file mode 100644
index 0000000..2b740bf
--- /dev/null
+++ b/.editorconfig
@@ -0,0 +1,19 @@
+root = true
+
+[*]
+charset = utf-8
+indent_style = space
+indent_size = 2
+trim_trailing_whitespace = true
+# Unix-style newlines with a newline ending every file
+end_of_line = lf
+insert_final_newline = true
+
+[*.{js,css,scss}]
+quote_type = single
+
+[*.{yml,yaml}]
+quote_type = double
+
[...]
```

All lines starting with a plus sign are lines that where added during this commit. And with that the exfiltrated data is shown.

## Conclusion
In commit 2aa...98de6 with comment "data exfiltration" data was found that was most likely sent to the remote repository `https://github.com/mennovanveenendaal/exfiltration_test.git`.

## Closing the investigation
Data from this device was successfully sent to a remote GitHub repository. Evidence of Git execution was found in the `Prefetch` files, which led to the discovery of the local repository folder and Git log files. These logs revealed the specific commits and the data that was pushed.

The goal of this investigation was to determine if there were traces of data exfiltration to GitHub on a Windows system. The results show that Windows does store information about Git usage, making it possible to track whether Git was used on the device.

While this case focused on a successful data exfiltration, it's important to consider what logs or evidence might appear if a Git push was blocked or failed. Additionally, had Git been uninstalled or files deleted after the exfiltration, **recovery options** should be explored, such as using forensic tools to recover deleted files or artifacts.

## Recommendations to Prevent Future Data Exfiltration
To prevent similar incidents, companies can implement the following strategies:

- **Network-level blocks** on Git or GitHub domains.
- **Device or network monitoring** to detect Git usage or abnormal traffic patterns.
- **Enforcing least privilege access** to prevent unauthorized installations of software like Git.





