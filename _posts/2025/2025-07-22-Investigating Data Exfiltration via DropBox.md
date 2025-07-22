---
title: "Investigating Data Exfiltration via Dropbox"
layout: post
categories: [Windows, Registry]
image:
  path: /assets/2025/cloud_data_exfil/Dropbox.png
  alt: exfiltration
---

After the data exfiltration investigation using [Git](https://www.mennovanveenendaal.com/posts/Data-Exfiltration-via-Git;-A-Forensic-Investigation.-Part-2,-Investigation/), someone asked whether it would be possible to determine what data was uploaded to Dropbox from an endpoint. To answer that question, I decided to investigate.

## Scenario
An alert was triggered by network monitoring, flagging a connection from a workstation to the Dropbox website. Since Dropbox can be used to upload files, the event was considered suspicious. Because Dropbox traffic is encrypted, the network logs didn’t show whether files were uploaded. However, based on the volume of data transferred, there was reason to suspect an upload occurred.

When questioned, the user gave vague answers. It was therefore decided to investigate the workstation further.

## Scope of Investigation
The investigation focused on a Windows 10 device from which the Dropbox connection was observed. The goals were:

- Determine whether any data was uploaded to Dropbox.
- Find out if any artifacts on the device could reveal what was uploaded.

The main questions I tried to answer were:
- [Was Dropbox installed on the device?](#was-dropbox-installed)
- [Was the Dropbox website visited from this device?](#was-dropbox-visited)
- [Are there traces of data being sent to DroBox?](#was-data-uploaded-to-dropbox)
- [If data was send, what data was sent,](#what-data-was-uploaded-to-dropbox)

## Workstation setup 
To simulate this scenario, I used a Windows 10 Enterprise 21H2 virtual machine. I created a folder named `secret_code` and downloaded the [Chirpy theme ZIP file](https://www.mennovanveenendaal.com/posts/Setting-up-the-Chirpy-theme/) to serve as sensitive content. The unzipped folder contained:
- 44 folders    
- 216 files
- 515,325 bytes (872,448 bytes on disk)

I then uploaded the folder `secret_code` to a newly created Dropbox directory via drag-and-drop in a browser.

At this point, the data was successfully exfiltrated to Dropbox.
![Dropbox](/assets/2025/cloud_data_exfil/upload.png)
_Fig.1 Dropbox upload_

![Code in Dropbox](/assets/2025/cloud_data_exfil/code_in_dropbox.png)
_Fig.2 Code in Dropbox_

## Data acquisition
Firsly I acquired the memory of the VM. 
```cmd
> cd C:\Program Files\Oracle\VirtualBox
> VBoxManage.exe list vms 

> VBoxManage.exe debugvm {VM-UUID} dumpvmcore --filename C:\Cases\cloud_data_exfiltration\win10-memory.raw

> certUtil -hashfile C:\Cases\cloud_data_exfiltration\win10-memory.raw > C:\Cases\cloud_data_exfiltration\win10-memory-hash.txt
```

To acquire the disk, I resumed the VM, turned if off and performed the disk acquasition
```cmd
> VBoxManage.exe list hdds 

> VBoxManage.exe clonemedium disk {disk-uuid} --format VHD win10-disk.vhd
```

The disk was mounted read-only, and I used my [.bat script](https://www.mennovanveenendaal.com/posts/Forensic-Investigation-setup-script/) to collect the forensic artifacts.

## General System Information
First I viewed the basic information of the machine, like the timezone settings, host name and user accounts.

### Timezone
```
>rip.exe -r SYSTEM -p timezone
Launching timezone v.20200518
timezone v.20200518
(System) Get TimeZoneInformation key contents

TimeZoneInformation key
ControlSet001\Control\TimeZoneInformation
LastWrite Time 2025-03-10 20:45:19Z
  DaylightName   -> @tzres.dll,-211
  StandardName   -> @tzres.dll,-212
  Bias           -> 480 (8 hours)
  ActiveTimeBias -> 420 (7 hours)
  TimeZoneKeyName-> Pacific Standard Time
```
- The `TimeZoneKeyName` subkey was set to `Pacific Standard Time`
- The ActiveTimeBias (the number of minutes offset from UTC) was 420. 420 divided by 60 gives an offset of 7, or **UTC-7**.
- The value of DaylightName was -211, translating to [Pacific Standard Time](https://www.nirsoft.net/dll_information/windows8/tzres_dll.html).

## Was Dropbox installed?
To rule out the use of the Dropbox application, I checked both UserAssist and registry entries. There were **no traces** of a Dropbox installation.

Registry analysis showed only Internet Explorer and Edge were installed. Internet Explorer had no recorded activity, so the focus shifted to Edge.

## Was Dropbox visited? 
The directory `Users\IEUser\AppData\Local\Microsoft\Edge\User Data\Default` contained Edge browser history.

![Edge browser data](/assets/2025/cloud_data_exfil/Edge_Browser_data.png)
_Fig.3 Edge browser data_

Using **DB Browser for SQLite**, I confirmed the URL `https://www.dropbox.com/home/test_upload/secret_code` was accessed. However, it did not indicate whether files were uploaded.

![urls](/assets/2025/cloud_data_exfil/visited_urls.png)
_Fig.4 Edge urls_

The timestamp 13386081378482240 and 13386083022649117 translated to:
```
10 Mar 2025 11:56:18 UTC and 10 march 2025 12:23:42 UTC
```
These match the earlier timestamps.

![urls](/assets/2025/cloud_data_exfil/epoch.png)
_Fig.5 Time Stamp_

## File Interaction Evidence
The Edge data showed the URL, but didn't show if, and which, files where uploaded. 
To find this I started looking at the files the user accessed during the timeframe 10 Mar 2025 11:56:18 UTC and 10 march 2025 12:23:42 UTC.

### ShellBags
The ShellBags Shellbags showed the locations browsed by the user.
`USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\Bags`

This contained entries for the folders `secret_code` and `\secret_code\jekyll-theme-chirpy-master`.

### Folder
I also found traces of the folder `secret_code` in `RecentDocs` registry key 
`NTUSER\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`:
```
----------------------------------------
recentdocs v.20200427
(NTUSER.DAT) Gets contents of user's RecentDocs key

RecentDocs
**All values printed in MRUList\MRUListEx order.
Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs
LastWrite Time: 2025-03-10 11:56:33Z
  2 = secret_code
  5 = jekyll-theme-chirpy-master.zip
  4 = This PC
  3 = Documents
  1 = The Internet
  0 = kglcheck/
```

### ComDlg32
`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32` showed Edge opened the folder `Local Documents\secret_code` on 10 March 2025 at 11:56:33 UTC.

![LastVisitedPidMRU](/assets/2025/cloud_data_exfil/LastVisitedPidMRU.png)
_Fig.6 Time Stamp_

### ActivitiesCache.db
In the ActivitiesCache Database I could see the application that was in focus. Around 2025-03-10 11:53:43 UTC the Edge browser was.
![ActivitiesCacheDB](/assets/2025/cloud_data_exfil/ActivitiesCacheDB.png)

### MFT
In the MFT records I filtered on the `secret_code` folder name, and found multiple enties for documents and folder. This confirmed that these where present. However, I coudn't determine the files that where actually uploaded.
![MFT](/assets/2025/cloud_data_exfil/MFT.png)

## Was data uploaded to Dropbox?
### Hindsight
Looking at the Edge browser data [figure 3](\#\#Edge\ Browser) there was a `sessions` folder. In the mounted image there was much more stored information from Egde.

Using [Hindsight](https://github.com/obsidianforensics/hindsight) I loaded the data from `Users\IEUser\AppData\Local\Microsoft\Edge\User Data\Default`. I exported the results to a sql database, and used DB Browser to view the data. The timeline table contained the entries that showed the dropbox URL:

![Hindsight](/assets/2025/cloud_data_exfil/hindsight_2.png)
_Fig.7 Dropbox URL's in Hindsight_

This showed the same data from the Edge history file, in a more ordenend way. `https://www.dropbox.com/home/test_upload/secret_code`. The timeline also showed the creation of the `test_upload` folder:

![Hindsight](/assets/2025/cloud_data_exfil/hindsight.png)
_Fig.8 Created folder_

### What data was uploaded to Dropbox?
In the storage table I found session storage data.

![Hindsight](/assets/2025/cloud_data_exfil/session_storage.png)
_Fig.9 Session Storage folder_

I filtered on `upload`, and found a upload event for 215 files:

```json
{
"@udcl:version": 4,
"detail": "{\"num_files\":215,\"upload_method\":\"chooser\",\"upload_id\":\"<ID>\",\"file_size\":1073,\"file_extension\":\"html\",\"batch_id\":\"batch-<ID>\",\"upload_destination\":\"/test_upload/secret_code/_layouts\",\"current_path\":\"/home/test_upload\",\"file_name\":\"archives.html\",\"last_progress_time\":0,\"percent_uploaded\":0,\"is_hanging_upload\":false,\"client_timestamp\":1741607809,\"client_timestamp_ms\":1741607809055,\"origin\":\"BROWSE\",\"uploader\":\"session_uploader\",\"default_drag_and_drop\":\"V2\",\"dropzone_area\":\"\",\"max_append_batch_size\":\"OFF\",\"batch_append\":\"false\",\"effectiveConnection\":\"4g\",\"is_encrypted\":\"false\",\"apiUsed\":\"session_api\",\"use_legacy_table\":true,\"@udcl:request_id\":\"<ID>\"}",
"id": "<ID>",
"name": "upload_dequeue",
"timestamp": 64286,
"type": "event"
}
```

The details in this:
- `num_files` = 215
- `file_size` = 1073
- `upload_destination` = `/test_upload/secret_code/[...]`
- `client_timestamp` = 1741607809 == 10 Mar 2025 11:56:49 (UTC 00:00)
- `file_name` = `archives.html`
- `session_uploader` = `default_drag_and_drop`

![Hindsight](/assets/2025/cloud_data_exfil/hindsight_upload.png)
_Fig.10 Data upload_

And following this, there where 528 more enties with the same information:
```json
[...]
{"type":"event","name":"upload_success","id":"<ID>","detail":"{\"num_files\":215,\"upload_method\":\"chooser\",\"upload_id\":\"<ID>\",\"file_size\":5838,\"file_extension\":\"html\",\"batch_id\":\"batch-<ID>\",\"upload_destination\":\"/test_upload/secret_code/_layouts\",\"current_path\":\"/home/test_upload\",\"file_name\":\"post.html\",\"last_progress_time\":0,\"percent_uploaded\":0,\"is_hanging_upload\":false,\"client_timestamp\":1741607813,\"client_timestamp_ms\":1741607812704,\"origin\":\"BROWSE\",\"uploader\":\"session_uploader\",\"default_drag_and_drop\":\"V2\",\"dropzone_area\":\"\",\"max_append_batch_size\":\"OFF\",\"batch_append\":\"false\",\"effectiveConnection\":\"4g\",\"is_encrypted\":\"false\",\"parallelCommit\":\"true\",\"timeInStep\":\"1359.6999999999534\",\"overallTime\":\"14832.5\",\"numBlocks\":\"1\",\"numChunks\":\"1\",\"retriedBlocks\":\"0\",\"retriedCommits\":\"0\",\"fullDuration\":\"14833\",\"preUploadDuration\":\"12469\",\"blockUploadWallDuration\":\"1004\",\"blockUploadThreadDuration\":\"551\",\"commitDuration\":\"1360\",\"fileSize\":\"5838\",\"effectiveUploadSpeed\":\"5814\",\"sizeNormalizedSpeed\":\"394\",\"fileMbps\":\"0.0031\",\"apiUsed\":\"session_api\",\"autoRetryCount\":\"0\",\"manualRetryCount\":\"0\",\"use_legacy_table\":true,\"@udcl:request_id\":\"<ID>\"}","@udcl:version":4,"timestamp":67934.40000000002},

{"type":"event","name":"upload_visually_complete","id":"<ID>","detail":"{\"num_files\":215,\"upload_method\":\"chooser\",\"upload_id\":\"<ID>\",\"file_size\":5838,\"file_extension\":\"html\",\"batch_id\":\"batch-<ID>\",\"upload_destination\":\"/test_upload/secret_code/_layouts\",\"current_path\":\"/home/test_upload\",\"file_name\":\"post.html\",\"last_progress_time\":0,\"percent_uploaded\":0,\"is_hanging_upload\":false,\"client_timestamp\":1741607813,\"client_timestamp_ms\":1741607812704,\"origin\":\"BROWSE\",\"uploader\":\"session_uploader\",\"default_drag_and_drop\":\"V2\",\"dropzone_area\":\"\",\"max_append_batch_size\":\"OFF\",\"batch_append\":\"false\",\"effectiveConnection\":\"4g\",\"is_encrypted\":\"false\",\"batchId\":\"batch-<ID>\",\"apiUsed\":\"session_api\",\"use_legacy_table\":true,\"@udcl:request_id\":\"<ID>\"}","@udcl:version":4,"timestamp":67934.5},

{"type":"event","name":"upload_success","id":"<ID>","detail":"{\"num_files\":215,\"upload_method\":\"chooser\",\"upload_id\":\"<ID>\",\"file_size\":441,\"file_extension\":\"html\",\"batch_id\":\"batch-z<ID>\",\"upload_destination\":\"/test_upload/secret_code/_layouts\",\"current_path\":\"/home/test_upload\",\"file_name\":\"page.html\",\"last_progress_time\":0,\"percent_uploaded\":0,\"is_hanging_upload\":false,\"client_timestamp\":1741607813,\"client_timestamp_ms\":1741607812704,\"origin\":\"BROWSE\",\"uploader\":\"session_uploader\",\"default_drag_and_drop\":\"V2\",\"dropzone_area\":\"\",\"max_append_batch_size\":\"OFF\",\"batch_append\":\"false\",\"effectiveConnection\":\"4g\",\"is_encrypted\":\"false\",\"parallelCommit\":\"true\",\"timeInStep\":\"1359.7999999999302\",\"overallTime\":\"14832.599999999977\",\"numBlocks\":\"1\",\"numChunks\":\"1\",\"retriedBlocks\":\"0\",\"retriedCommits\":\"0\",\"fullDuration\":\"14833\",\"preUploadDuration\":\"12442\",\"blockUploadWallDuration\":\"1030\",\"blockUploadThreadDuration\":\"594\",\"commitDuration\":\"1360\",\"fileSize\":\"441\",\"effectiveUploadSpeed\":\"428\",\"sizeNormalizedSpeed\":\"30\",\"fileMbps\":\"0.0002\",\"apiUsed\":\"session_api\",\"autoRetryCount\":\"0\",\"manualRetryCount\":\"0\",\"use_legacy_table\":true,\"@udcl:request_id\":\"<ID>\"}","@udcl:version":4,"timestamp":67934.59999999998},

{"type":"event","name":"upload_visually_complete","id":"<ID>","detail":"{\"num_files\":215,\"upload_method\":\"chooser\",\"upload_id\":\"<ID>\",\"file_size\":441,\"file_extension\":\"html\",\"batch_id\":\"batch-<ID>\",\"upload_destination\":\"/test_upload/secret_code/_layouts\",\"current_path\":\"/home/test_upload\",\"file_name\":\"page.html\",\"last_progress_time\":0,\"percent_uploaded\":0,\"is_hanging_upload\":false,\"client_timestamp\":1741607813,\"client_timestamp_ms\":1741607812704,\"origin\":\"BROWSE\",\"uploader\":\"session_uploader\",\"default_drag_and_drop\":\"V2\",\"dropzone_area\":\"\",\"max_append_batch_size\":\"OFF\",\"batch_append\":\"false\",\"effectiveConnection\":\"4g\",\"is_encrypted\":\"false\",\"batchId\":\"batch-<ID>\",\"apiUsed\":\"session_api\",\"use_legacy_table\":true,\"@udcl:request_id\":\"<ID>\"}","@udcl:version":4,"timestamp":67934.70000000007},

{"type":"event","name":"upload_append_success","id":"<ID>","detail":"{\"num_files\":215,\"upload_method\":\"chooser\",\"upload_id\":\"<ID>\",\"file_size\":7303,\"file_extension\":\"md\",\"batch_id\":\"batch-<ID>\",\"upload_destination\":\"/test_upload/secret_code/_posts\",\"current_path\":\"/home/test_upload\",\"file_name\":\"2019-08-09-getting-started.md\",\"last_progress_time\":0,\"percent_uploaded\":0,\"is_hanging_upload\":false,\"client_timestamp\":1741607809,\"client_timestamp_ms\":1741607809387,\"origin\":\"BROWSE\",\"uploader\":\"session_uploader\",\"default_drag_and_drop\":\"V2\",\"dropzone_area\":\"\",\"max_append_batch_size\":\"OFF\",\"batch_append\":\"false\",\"effectiveConnection\":\"4g\",\"is_encrypted\":\"false\",\"parallelCommit\":\"true\",\"timeInStep\":\"1098.0999999999767\",\"apiUsed\":\"session_api\",\"use_legacy_table\":true,\"@udcl:request_id\":\"<ID>\"}","@udcl:version":4,"timestamp":64617.59999999998},

{"type":"event","name":"upload_append_success","id":"<ID>","detail":"{\"num_files\":215,\"upload_method\":\"chooser\",\"upload_id\":\"<ID>\",\"file_size\":6090,\"file_extension\":\"md\",\"batch_id\":\"batch-<ID>\",\"upload_destination\":\"/test_upload/secret_code/_posts\",\"current_path\":\"/home/test_upload\",\"file_name\":\"2019-08-08-text-and-typography.md\",\"last_progress_time\":0,\"percent_uploaded\":0,\"is_hanging_upload\":false,\"client_timestamp\":1741607809,\"client_timestamp_ms\":1741607809387,\"origin\":\"BROWSE\",\"uploader\":\"session_uploader\",\"default_drag_and_drop\":\"V2\",\"dropzone_area\":\"\",\"max_append_batch_size\":\"OFF\",\"batch_append\":\"false\",\"effectiveConnection\":\"4g\",\"is_encrypted\":\"false\",\"parallelCommit\":\"true\",\"timeInStep\":\"1175.4000000000233\",\"apiUsed\":\"session_api\",\"use_legacy_table\":true,\"@udcl:request_id\":\"<ID>\"}","@udcl:version":4,"timestamp":64617.59999999998},

{"type":"event","name":"upload_append_success","id":"<ID>","detail":"{\"num_files\":215,\"upload_method\":\"chooser\",\"upload_id\":\"<ID>\",\"file_size\":1970,\"file_extension\":\"md\",\"batch_id\":\"batch-<ID>\",\"upload_destination\":\"/test_upload/secret_code/_posts\",\"current_path\":\"/home/test_upload\",\"file_name\":\"2019-08-11-customize-the-favicon.md\",\"last_progress_time\":0,\"percent_uploaded\":0,\"is_hanging_upload\":false,\"client_timestamp\":1741607809,\"client_timestamp_ms\":1741607809387,\"origin\":\"BROWSE\",\"uploader\":\"session_uploader\",\"default_drag_and_drop\":\"V2\",\"dropzone_area\":\"\",\"max_append_batch_size\":\"OFF\",\"batch_append\":\"false\",\"effectiveConnection\":\"4g\",\"is_encrypted\":\"false\",\"parallelCommit\":\"true\",\"timeInStep\":\"1063.4000000000233\",\"apiUsed\":\"session_api\",\"use_legacy_table\":true,\"@udcl:request_id\":\"<ID>\"}","@udcl:version":4,"timestamp":64617.80000000005},


[...]
```

In the above snippet the details of 5 files are listed. 
- post.html
- page.html
- 2019-08-09-getting-started.md
- 2019-08-08-text-and-typography.md
- 2019-08-11-customize-the-favicon.md

With this the names and details of the uploaded files are found.

## Conclusion
The session storage data confirmed that files were uploaded to Dropbox including the filenames and sizes. This provides evidence of data exfiltration via a browser session. Based on this, further investigation could reviel the data in the files.

