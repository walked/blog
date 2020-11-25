---
title: "Quick Tidbit: Download and Install MSI with PowerShell"
date: 2015-08-09T12:32:30-05:00
draft: false
tags: ["powershell"]
---

This one comes via needing to deploy via GFI Remote Management and DropBox. This scenario was needed for a deployment of a VPN client in a GFI managed environment where there was no Active Directory present. Very simply put; PowerShell pulls from a public DropBox link, and silently installs.
<!--more-->

```powershell
(New-Object System.Net.WebClient).DownloadFile("https://www.dropbox.com/s/abcxyz/FILE.MSI?dl=1", "C:\Windows\Temp\example.msi")

##The dl=1 is necessary for this dropbox link; otherwise it's processed as a landing page with a file summary and a download link.

$MSIFileSize = 100 
#This is the known complete size of the MSI file; prevents attempted execution prior to complete download

do{
    Start-Sleep -Seconds 2
    $FileSize= (Get-Item C:\Windows\Temp\test.msi).Length
} until ($FileSize -eq $MSIFileSize)

## Be sure to check the MSI flags applicable to your MSI
Msiexec /i C:\Windows\Temp\test.msi /norestart /passive /qn /lvx* C:\Windows\Temp\logtest.txt
```