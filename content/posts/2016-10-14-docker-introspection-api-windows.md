---
title: "Docker on Windows 2016; Container Introspection via Docker Daemon / API"
date: 2016-10-14T12:32:30-05:00
draft: false
tags: ["docker", "windows"]
---

So with the RTM of Windows Server 2016, with it comes official support for docker; and with that - containers. Awesome. In this post, we'll talk about container introspection, via the Docker Daemon / API. First and foremost, I'm not going to get into the nitty gritty of installing and configuring docker.

<!--more-->
 [Microsoft/Docker have done a decent job of that already](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/quick_start/quick_start_windows_server). So, what we are going to talk about is setting up the Docker daemon to allow us to do some introspection from within a container. **NOTE:** This is not a hardened solution; this is just a lab environment configuration; you would want to enable TLS and ensure your ACLs/firewall are configured correctly in practice. So lets talk about getting the daemon running and listening. It's pretty simple, but not readily obvious or particularly well documented _for Windows_. We're doing to need to take a few steps beyond the basic Docker install to get the Daemon responding.



1. Open the firewall on the docker host.

    ```
    netsh advfirewall firewall add rule name="Docker daemon API" dir=in action=allow protocol=TCP localport=2375
    ```

2. Configure the file at C:\programdata\docker\config\daemon.json (this did not exist by default on my installation)

    ```powershell
    new-item -Type File c:\ProgramData\docker\config\daemon.json
    ```

3. Edit the contents of daemon.json to:

    ```
    { "hosts": ["tcp://0.0.0.0:2375", "npipe://"] }
    ```

    This causes the daemon to listen on all IP addresses; both externally facing, and internally facing for container networking.

4. Restart Docker

    ```
    Restart-Service docker
    ```

Cool. Now what do we do with it? Let's look at the [Docker API documentation.](https://docs.docker.com/engine/reference/api/docker_remote_api_v1.24/) Hm. Ok. So, from within a docker container, to learn about itself, we'll run:

```powershell
$containerInfo = Invoke-WebRequest 1.2.3.4:2375/containers/HEXCONTAINERIDHERE/json -UseBasicParsing | ConvertFrom-Json
#Note: 1.2.3.4 would be the gateway IP address of the Container; note there is an internal virtual network and you must use the internal interface of the docker host
#In this example you can just run _ipconfig_ and look at the gateway; use that.
```

Let's see what we have here:

```
Id              : 7729e5f666261517c3f7da4c5951c61a1b80178dd0da4517d37069879bcda01f
Names           : {/tiny_mirzakhani}
Image           : microsoft/iis
ImageID         : sha256:6e30590a2139185500136f0051a36d7b390e649a96dc51d3a3477eff08d69fbd
Command         : C:\ServiceMonitor.exe w3svc
Created         : 1476365755
Ports           : {}
Labels          :
State           : running
Status          : Up 56 minutes
HostConfig      : @{NetworkMode=default}
NetworkSettings : @{Networks=}
Mounts          : {}
```

Awesome. If I had exposed ports; I'd be able to see which here. If I needed to know the Image it was running, it would also be here. This is a short post, but it's here because the documentation on getting / using the API is pretty much non-existent for Windows Server 2016 and it's really handy stuff. Stay tuned for getting this running in tandem with a Flask app for automation.