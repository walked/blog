---
title: "SCCM OSD Naming"
date: 2015-08-20T12:32:30-05:00
draft: false
tags: ["sccm"]
---

So I've been working through the process of putting together a new Windows 8.1 image for OSD Deployment within our organization. 

<!--more-->

One item that's always irked me a bit has been computer naming having no integration within SCCM. Either way, here's a quick script process to rename a PC as a part of a build process. First and foremost, we need to get a SecureString variant of the password we want to actually save somewhere; we wouldnt want it plaintext in any context. This still has some risks, but if the shares where the SCCM package is held is secured, and the account at hand has limited rights, it's not too bad. First off - lets get that PowerShell Secure String - do this at a command prompt - don't include this plaintext in the script...please:


```powershell
"PASSWORD" | ConvertTo-SecureString -AsPlainText - Force | ConvertFrom-SecureString | Out-File "C:\Temp\savedpwd.txt"
```

Once we have this text string we can either reference the text file in our scripts or use the contents as a part of a PowerShell Credential object:

```powershell
#We use a serialnumber object from the Dell BIOS to insert into our PC naming convention
$sn = gwmi win32_bios | Select serialnumber
$NewComputerName = "PC-PREFIX" + $sn.serialnumber 

#Specify the user credentials
#Note! The password string is the text that was output in the previous command inside savedpwd.txt; I kept it inclusive here but you can also read-file the text file
$user = "domain\account"
$pass = "01000sadfasdfasdf67f40920000000004800000a0000000100000000551ae1296831a0af8f89d2cf2fe50ac200000asdf089o372hnioujhda0f98h2lkjshodc0f89234r9c9f9b140000009fa7413ae386f1c47f7cb4dfe09750cf3d785fed"

#Create the credential object for the Rename-Computer command
$cred = new-object -TypeName System.Management.Automation.PSCredential -ArgumentList $user, ($pass | ConvertTo-SecureString)

#Go!
Rename-Computer -newname $NewComputerName -DomainCredential $cred  -Restart
```

Bam; create a package in SCCM you can reference in the Task Sequence; and you're good to go. You can use whatever WMI queries you need in order to get your computer naming convention down; if we were deploying in multiple sites we could also use the subnet to use a site-code-prefix; however in our case we dont deploy user workstations at more than one site.