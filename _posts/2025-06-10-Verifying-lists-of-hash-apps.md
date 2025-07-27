---
layout: post 
title: "Verifying Android apps in VirusTotal with their hashes" 
categories: [android, forensics] 
tags: [android, forensics, virustotal, vt, hash, collections, collections, malware] 
author: ZoqueLabs 
description: Mini-tutorial to investigate lists of app hashes in Virus Total 
lang: en 
---

<div align=center>
<h1>--[ Checking many Android apps on VirusTotal with their hashes ]--</h1>
By: ZoqueLabs
<br>
<br>
This writing is distributed under a Creative Commons CC BY-SA (Attribution-ShareAlike) license .
<br>
<a href="/android/forense/2025/06/10/Verificando-listas-hash-apps.html">spanish version</a>
</div>


## Intro 

This mini tutorial is intended to help you check if a list of applications installed on an Android device has been reported as malicious, using its hash (a kind of unique fingerprint of the file). 

Tools like **Colander** or **[MVT](https://docs.mvt.re/en/latest/)** (Mobile Verification Toolkit) allow you to do this verification, but they do it one by one. If you only have a few hashes, it can work. But if you are working with many —like when doing a full forensic extraction with `androidqf`— this process becomes very slow and quickly consumes the VirusTotal API limit. 

In this tutorial we show how to do this verification in bulk, that is, review several hashes at the same time, saving time and energy. 

We start from a forensic extraction of the Android device generated with `androidqf`, which includes a file called `packages.json` where the SHA256 hashes of the installed applications are found. You can also use `hashes.csv` if available, but we're primarily focusing on `packages.json` here because that's what's usually present even in more basic extractions. 

The idea is to help you identify suspicious or outright malicious behavior in apps on the device, without having to go through them one by one. 

--- 

## Preparing the Environment with androidqf 

To get started, we'll need to perform a forensic extraction of the Android device using the [androidqf](https://github.com/mvt-project/androidqf) tool. You'll then have a folder with several files, including the all-important `packages.json`. 

```bash
$ ls
acquisition.json  dumpsys.txt  logcat.txt          selinux.txt          tmp
apks              env.txt      logs                services.txt
backup.ab         files.json   packages.json       settings_global.txt
bugreport.zip     getprop.txt  processes.txt       settings_secure.txt
command.log       hashes.csv   root_binaries.json  settings_system.txt
```
&nbsp;

## Extracting App Hashes from `packages.json` 

The `packages.json` contains detailed information about all the applications installed on the device, including the cryptographic hashes of each apk and its subcomponents.

```bash 
 { 
        "name": "com.android.pacprocessor", 
        "files": [ 
            { 
                "path": "/system/app/PacProcessor/PacProcessor.apk", 
                "local_name": "", 
                "md5": "95ed855e694de1ad40e4d3500a24952f",
                "sha1": "e40cd26f03becf29a4f887978ea57f21c7e30cc4", 
                "sha256": "946cb5797f9a26a93709a7f01175b890bc010ba80182f8847ede24a39c9b9660", 
                "sha512": "77c2fb531da7b2751e3abb9ef25c3d715e75d3978301ee00b66edb3c171205fa97d7daa3fb21c2725aece9d7392b7a7ad90da58f1ea64520398ccebf0c3a6d67", 
``` 
&nbsp;
To extract the complete list of SHA256 hashes ​​of your apps from `packages.json`, we'll use the Linux terminal (the process might be similar on MacOS or Windows). 

However, in this case we are only interested in applications that could be suspicious — that is, those that are not native to the system or were installed from unknown sources — for analysis.

Now then, let's go...

First, navigate to the directory where the extraction is located: 

```bash 
cd path/to/your/backup/ 
``` 
&nbsp;
If you want to extract all the hashes, you can run this command: 
```bash
jq -r '.[].files.[].sha256' packages.json 
``` 
&nbsp;
But since our goal is to focus on "suspicious" applications, we are going to use more specific filters with this command:
```bash
jq -r '.[] | select (((.name | startswith("com.google.android.") or startswith("com.android.") or startswith("com.samsung") or startswith("com.sec.")) and (.system == true)) | not)' packages.json | jq -r '.files.[].sha256'
```
&nbsp;
**What does this command do and what is our plan?**

This command uses `jq` (a command-line tool to process JSON files) to process the `packages.json` file and extract only the SHA256 hashes of applications that do not appear to be system apps or from well-known manufacturers such as Google or the device manufacturer —in our case, Samsung, but you can replace this with your own-.

So, let's break it down:

1. `select(...)` applies a filter to exclude:

  - Apps whose name starts with `com.google.android.`, `com.android.`, `com.samsung` or `com.sec`.
  - And also are marked as system `(.system == true)`
  - `| not` inverts the condition so we are left only with those that do not meet that pattern, that is, apps installed from unknown sources.
2. `.files[].sha256` extracts the hash of each file associated with those filtered apps.

Thus, by running this command you will obtain a filtered list of applications more interesting for analysis with VirusTotal.

```bash 
$ jq -r '.[] | select (((.name | startswith("com.google.android.") or startswith("com.android.") or startswith("com.samsung") or startswith("com.sec.")) and (.system == true)) | not)' packages.json | jq -r '.files.[].sha256'
5f4144359f8fdf52e6d4471a11438aa2c209e4af2d8bc90a4111975d1227c0aa 
9948eaa76138f8a45943cdd81838e4a053a2734ef8a326e108dc3b0b5c4f409d 
5014e20fb03bca12d456f278faf2b1f2f43326930c062e0705fb2cebedfe23b2 
... more hashes 
... more hashes 
``` 
&nbsp;
If you want to save the list of hashes in a file and then upload it to VirusTotal, you can do it like this: 
```bash 
jq -r '.[] | select (((.name | startswith("com.google.android.") or startswith("com.android.") or startswith("com.samsung") or startswith("com.sec.")) and (.system == true)) | not)' packages.json | jq -r '.files.[].sha256' > hashesobtenidos.txt 
``` 
&nbsp;
With this file you will have all the hashes in bulk. 

--- 
## Create a collection in VirusTotal with the extracted hashes 

Once we have the list of hashes of the suspicious applications, we can use the _collections_ functionality of VirusTotal to analyze them together.

In this tutorial we use the free version of VirusTotal, because the idea is to provide accessible resources without relying on paid licenses.

However: in the free version, when you upload many hashes to a collection, the result filtering is not as friendly. You cannot, for example, easily sort by number of detections. That’s why it’s important that before uploading, you’ve already carefully filtered the apps that could truly be suspicious, just as we did in the previous step.

If you have access to a VirusTotal premium account, that _bulk_ analysis will be much smoother, but in this case, we work with what we have.

### What are VirusTotal collections and why use them? 

The [_collections_](https://blog.virustotal.com/2021/11/introducing-virustotal-collections.html) section in VirusTotal is very useful because it allows us to gather in one place several indicators related to the same analysis or case, such as hashes, URLs, domains or IP addresses. We can update these collections with new information, easily share them with others via a link, and jointly review the metadata provided by VirusTotal, such as the antivirus engines that detect threats or the labels that describe their behavior. This makes the analysis clearer, more organized, and more collaborative, especially when working in teams or needing to document findings for reports.

**Create a collection step by step** 

1. Enter VirusTotal, create an account or log in if you already have one. 
2. You can click on the search icon in VT or click [here](https://www.virustotal.com/gui/collections). 

 
![Image VT](/assets/vt1.png) 
 

3. Click on **Create new collection**. 

 
![Image VT](/assets/vt2.png) 
 

4. Assign a name to your collection pj: `hashapps`. 
5. Paste the hashes you extracted from `packages.json`, one per line. 
6. Save the collection. 

## What data does the VirusTotal analysis provide? 

VirusTotal takes care of analyzing the hashes automatically and shows you several important data to review the applications. 

 
![Image VT](/assets/vt3.png) 
 

**Antivirus engine detections**: Tells you if an engine has flagged the file as malicious, and what name it gives it (for example, `Android/TrojanSpy.Agent`). 

**Descriptive labels**: These labels not only indicate whether it is a virus, but also point out other behaviors or characteristics, such as: 
  - `has-trackers`: Whether the app includes known trackers (such as Google, Facebook, etc.). 
  - `uses-permission`: Whether it uses sensitive permissions such as access to SMS, camera, location, or storage. 
  - `obfuscator`: Whether the code is obfuscated, which may indicate that it is trying to hide something. 
  - Other labels such as `spyware`, `dropper`, `apk-downloader`, among others, can help you better understand the app's behavior. 


**Other file information**: 
  - File size and type. 
  - Date scanned. 
  - File name (if kept). 
  - Certificate or digital signature information, if available. 
  - Technical details of the `AndroidManifest.xml`, such as the permissions and services it declares. 

**Relationships with other elements**: VirusTotal can also show if that hash is linked to other files, URLs, domains, or known campaigns. 

All of this allows for a more complete review of whether an application poses a risk, even when it is not directly classified as malware. 

### Hashes ready 

And that's it. With these steps, you now have a quick and easy way to verify Android applications. We hope you enjoy this mini-tutorial. 
