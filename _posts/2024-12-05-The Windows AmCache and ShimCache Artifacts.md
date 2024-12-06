---
title: "The Windows AmCache and ShimCache Artifacts"
layout: post
categories: [Windows, Registry]
image:
  path: /assets/2024/amcache/amcache.png
  alt: AmCache and ShimCache
---
In my previous [article](https://www.mennovanveenendaal.com/posts/Data-Exfiltration-via-Git;-A-Forensic-Investigation.-Part-2,-Investigation/), I investigated whether sensitive data was exfiltrated using Git. During this investigation, I discovered that **AmCache** contained multiple entries indicating that the application `Git.exe` was executed on the system. In this article, we’ll explore two critical Windows artifacts, **AmCache** and **ShimCache**, which provide valuable forensic insights. 

These artifacts can help determine if programs were executed on a system, where they were launched from, and when they were accessed. Understanding and analyzing these artifacts can be pivotal in uncovering malicious activity or reconstructing user behavior.


## What Is AmCache?
The **AmCache** is a registry hive (`Amcache.hve`) in Windows used to track programs that are installed or executed on a system. It contains detailed metadata, such as:

- The full path of the executable
- The SHA1 hash of the program
- Timestamps (e.g., installation and execution times)
- File size
- [DLLs](https://learn.microsoft.com/en-us/troubleshoot/windows-client/setup-upgrade-and-drivers/dynamic-link-library) (Dynamic Link Libraries) used by the program.

Windows uses this data to optimize program execution and load times.

### Where Is AmCache Located?

The AmCache hive is stored at the following path: `C:\Windows\AppCompat\Programs\Amcache.hve`.

![AmCache](/assets/2024/amcache/ftk.png)
_Fig.1 AmCache hive in FTK_

### Why Is AmCache Valuable for Forensic Investigations?
The AmCache holds forensic value because it tracks important details about program execution, even if the program or file has been [deleted from the system](https://cdn-dynmedia-1.microsoft.com/is/content/microsoftcorp/microsoft/final/en-us/microsoft-brand/documents/IR-Guidebook-Final.pdf). Key details include:

- **Execution Tracking:** It logs when and where an application was executed, along with the full file path.
- **SHA1 File Hashes:** These hashes can:
    - **Verify file integrity:** If the current file hash differs from the hash in AmCache, it may indicate the file was modified (potentially maliciously) after its first execution.
    - **Identify unknown or malicious files:** Hashes can be cross-referenced with threat intelligence databases to flag suspicious programs and identify potential indicators of compromise.

## Tools for Parsing AmCache Data
### 1. Registry Explorer
[Eric Zimmerman's Registry Explorer](https://ericzimmerman.github.io/#!index.md) is a powerful tool to manually examine, in this case, the AmCache hive. It allows you to view the registry’s contents and incorporate transaction logs, which capture recent changes not yet committed to the main registry file.

![Registry Explorer](/assets/2024/amcache/registry_explorer.png)
_Fig.2 AmCache in Registry Explorer_

Registry Explorer also enables parsing of **transaction logs**, which record changes before they are committed to the registry. This is useful for recovering deleted or modified records, even when the registry itself is locked.

### 2. AmcacheParser
[Eric Zimmerman's AmcacheParser](https://github.com/EricZimmerman/AmcacheParser) automates the extraction of AmCache data into a CSV formatted file. Here's an example of its usage:
```shell
AmcacheParser -f "Amcache.hve" -i --csv "Amcache\"
```
**Parameters:**
- **`-f`**: Input file
- **`-i`**: Include program entries
- **`--csv`**: Output directory for saving CSV results

![AmcacheParser output](/assets/2024/amcache/parser_output.png){: w="500"}
_Fig.3 Output of AmcacheParser_

You can then further analyze the CSV output in tools like [Timeline Explorer](https://ericzimmerman.github.io/#!index.md).

![Timeline explorer](/assets/2024/amcache/timeline_explorer.png)
_Fig.4 Timeline explorer_

### 3. RegRipper
[RegRipper](https://github.com/keydet89/RegRipper3.0) is another popular tool for extracting key information from the registry. It generates detailed reports based on predefined plugins.

![RegRipper](/assets/2024/amcache/regripper.png){: w="500"}
_Fig.5 RegRipper GUI_

Sample output:
```text
c:\windows\systemtemp\google7260_1898517602\bin\updater.exe  LastWrite: 2024-10-11 18:48:46Z
Hash: 3d824f6aa75611478e56f4f56d0a6f6db8cb1c9b

c:\windows\systemtemp\chrome_unpacker_beginunzipping1616_869322099\129.0.6668.101_chrome_installer.exe  LastWrite: 2024-10-11 18:48:55Z
Hash: 57a36678d556042be81fa093d9a3b2c5921bd917
```

## ShimCache
The **Application Compatibility Cache (ShimCache)** stores information about application compatibility with the current version of the operating system. Its primary purpose is to help Windows manage compatibility settings, but it also logs the **name, location, and last modification time** of executables.

This information is written to the kernel memory, and written to the memory during a clean shutdown/restart. The data may be incomplete after a crash or power loss of the system.

Unlike AmCache, it does not confirm execution but instead shows **program presence** on a system.

### Location
The ShimCache data is located in the SYSTEM hive:  
`SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache`.

### ShimCache’s Forensic Value
While the ShimCache provides valuable information about file presence, it has an important limitation:  it does not confirm the [execution of the application](https://cdn-dynmedia-1.microsoft.com/is/content/microsoftcorp/microsoft/final/en-us/microsoft-brand/documents/IR-Guidebook-Final.pdf). This is in contrast to AmCache, which logs execution events. ShimCache only indicates that an executable existed at some point on the system.

The ShimCache can corroborate findings from other artifacts like Prefetch files, Event Logs, or network logs, providing a more complete view of system activity.

## Tools for Analyzing ShimCache
### 1. Registry Explorer:
After loading the SYSTEM hive into Registry Explorer you can export the ShimCache data as a `.bin` file and examine it in hex editors like WinHex.

![ShimCache](/assets/2024/amcache/ShimCache.png)
_Fig.6 ShimCache in Registry Explorer_

In WinHex we can read the filepaths easily, the rest of the data isn't translated by default (the `10ts` is recurring and could be some header) .

![WinHex](/assets/2024/amcache/ShimCache.png)
_Fig.7 ShimCache.bin in WinHex_

### 2. AppCompatCacheParser:
[Eric Zimmerman's AppCompatCacheParser](https://github.com/EricZimmerman/AppCompatCacheParser) can extract ShimCache data into a readable CSV format:

```shell
AppCompatCacheParser.exe -f SYSTEM --csv SimCache
```
- **-f** input file
- **--csv** output directory to save the CSV formatted result to 

AppCompatCacheParser output:
![AppCompatCacheParser](/assets/2024/amcache/appcompatcacheparser.png)
_Fig.8 AppCompatCacheParser output CSV file_

### 3. Volatility plugin ShimCache
If analyzing memory dumps, the [ShimCacheMem plugin](https://github.com/mandiant/Volatility-Plugins/tree/master/shimcachemem) can extract ShimCache data from volatile memory:

```shell
volatility -f dump.mem --profile= shimcachemem
```

## Conslusion
The AmCache and ShimCache are a great source for finding indications of program presence and execution. While AmCache confirms execution and provides hashes for integrity checks, ShimCache is useful for identifying files present on a system. Combined, these artifacts give information regarding system activity and can be useful in malware investigations or user activity analysis.

However, Forensic analysis should never rely solely on AmCache or ShimCache. To build a robust timeline or confirm execution, investigators should cross-reference these artifacts with:

- **Prefetch files:** To confirm execution and track file paths.
- **Event Logs:** To identify associated user activity or errors.
- **Network logs:** To track any external communication by suspicious executables.


