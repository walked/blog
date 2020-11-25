---
title: "cDscDocker 1.2.0 - Now With Daemon Support"
date: 2016-11-16T12:32:30-05:00
draft: false
tags: ["powershell", "docker", "windows"]
---

Woohoo; now we're cooking with fire. The cDscDocker resource has been updated to include giving the ability to expose the docker daemon on an interface address of your choice. 

<!--more-->

_Sidebar: this was insanely frustrating as docker is picky about text encoding in daemon.json, and PowerShell Out-File does not support UTF8 encoding without BOM. Fun times trying to troubleshoot why docker suddenly wouldnt start._ 

Either way, the links: 

[PowerShell Gallery](https://www.powershellgallery.com/packages/cDscDocker/1.2.0) 
[GitHub](https://github.com/walked/cDscDocker) 

### A few notes to follow:

Known quirks:

*   If you run this with docker installed, but a broken service it will fail on LCM test as it attempts to run commands against docker. I'll be doing some error handling for this later; but for now - if your docker service is errored out or unexpectedly stopped, you're probably in for a bad time.
*   You need to suppoly a boolean value to the exposeApi property AND a daemonInterface value in the format of "tcp://0.0.0.0:2375"; I will probably make the exposeApi property mandatory on the next minor release.

  Example:

```powershell
Configuration dockerConfig
{
    Import-DscResource -module cDscDocker

    Node localhost{

    LocalConfigurationManager
    {
        RebootNodeIfNeeded = $true
    }
        cDscDocker test
        {
            docker = "docker" #key; mandatory
            Ensure = 'Present' #mandatory field (Available options: Present, Absent)
            swarm = 'Active' #mandatory field (Available options: Active, Inactive)
            swarmToken = "SWMTKN-1-guidguidguidguidguidguidguidguid" #your swarm token here
            swarmURI = "192.168.1.100:2377" #IP address of manager node
            exposeApi = $true #this is a boolean value; not an enum like the the Ensure ["Present","Absent"] option above
            daemonInterface = 'tcp://0.0.0.0:2375' #must be full URI format mimicing a daemon.json key-value

        }
    }
}

dockerConfig
```

Full source on [GitHub](https://github.com/walked/cDscDocker).   

If you run into any quirks, bugs, or otherwise have a suggestion - reach out. The next release will likely be mostly refactoring a bit and improving error checking for oddities like errored docker service states.