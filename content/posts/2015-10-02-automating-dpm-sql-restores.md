---
title: "Automating DPM Restores for SQL Server DB"
date: 2015-10-02T12:32:30-05:00
draft: false
tags: ["sql"]
---

Scenario: The development team needs periodic copies of all databases in a protection group distributed to a share across a site link.

<!--more-->

 A combination of a DPM scheduled task and DFS replication gets this sorted out. There's actually quite a lot of moving parts to this, but I did not want to get into running separate SQL dumps when DPM is already copying full database sets (in fact, I've had a developer truncate logs and entirely break a DPM protection group. Fun times!) Either way, let's get to the script:



```powershell 
$SERVER       = "DPM-SERVER"
$PGName       = "PROTECTION_GROUP-NAME"
$TargetServer = "DESTINATION_SERVER"
$RESTOREPATH  = "D:\Pending"  #This is the full path on the destination server; no relative paths here

$PG = Get-ProtectionGroup -DPMServerName $server | Where-Object {$_.name -eq  $PGName}
$DS = Get-DPMDatasource -ProtectionGroup $PG[0]

foreach ($store in $DS)
{
    $RecoveryObject = Get-DPMRecoverypoint -Datasource $store | Sort -Property RepresentedPointInTime -Descending | Select-Object -First 1
    $RecoveryOption =  New-DPMRecoveryOption -SQL -TargetServer $TargetServer -RecoveryLocation CopyToFolder -RecoveryType Restore -TargetLocation $RESTOREPATH
    Restore-DPMRecoverableItem -RecoverableItem $RecoveryObject -RecoveryOption $RecoveryOption
}

#This loop serves to monitor for the restores to complete.
do{
    start-sleep -s 90
    $a = Get-DPMJob -ProtectionGroup $PG -Status InProgress
}
while($a.length -gt 0)

# Once the restore is entirely completed, it allows us to kick off a scheduled 
# task to clean up the horrendous directory structure left by DPM; this is not mandatory but really helps.
schtasks /run /s "$TargetServer" /tn "WHATEVER_CLEANUP_SCHTASK"</pre>

For such a simple script; it was surprisngly frustrating to get this operating the way I'd like. Dang, man, DPM's automation capabilities are a bit annoying. But it works; so that's good enough. Note: the cleanup script can be whatever you want; the /tn flag references whatever you name the scheduled task on the target server. For some inspiration, here's the scrubbed cleanup script I'm using

<pre class="toolbar-overlay:false lang:ps decode:true  ">$sourcedir = "D:\source" 
$destdir   = "D:\dest"

#You will need to confirm 7zip presence on scripted host;
#set-alias 7z "$env:PROGRAMW6432\7-zip\7z.exe" ##This command works for installed 7zip rather than console application
set-alias 7z "C:\7za\7za.exe"

$today     = Get-Date -Format yyyyMMdd

#Create today's date directory
New-Item -Path $destdir\$today -ItemType Directory -ErrorAction SilentlyContinue

$files = Get-ChildItem $sourcedir -Recurse -File
foreach ($file in $files)
{
    $name = $file.name
    $directory = $file.DirectoryName
    7z a -t7z "$directory\$file.7z" "$directory\$name"
}

Get-ChildItem $sourcedir -Recurse -File -Filter "*.7z"   | Move-Item -Destination $destdir\$today

## ONLY relevant if you want to clean up source directory
Get-ChildItem $sourcedir -Recurse -Directory | Remove-Item -Force -Recurse
```