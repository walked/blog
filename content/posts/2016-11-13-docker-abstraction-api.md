---
title: "Server 2016 - Docker Abstraction API with Flask"
date: 2016-11-13T12:32:30-05:00
draft: false
tags: ["docker", "python"]
---

So; lets talk about a docker host on Windows Server 2016, and building an abstraction API with Python/Flask. This will be a multi-part post as my time permits. 

<!--more-->

The basic scenario is that a team may have a need to share a docker server, with standardized docker images, but with needs not addressed directly by the docker daemon API. 

First up; in a previous post I addressed setting up Windows Server 2016 to listen to docker requests externally.  _(note: I know this is a post about introspection, but this will also cover you for allowing access on external interfaces)_   !

 So, first off, why? Many reasons at hand:

*   To play "traffic cop" - in this case we have multiple users that need to instantiate environments (containers) but without being able to run into infinity (on purpose or by accident)
*   It can perform authentication against whatever authentication provider we want to use; so long as Flask supports it (or you can build it in!)
*   We can do additional metrics and environment management outside of what the docker API can to (e.g. simple orchestration simplification)
*   Simplification for our team; no need to get nitty gritty

Additional factors:

*   We are in an environment which dictates no local admin rights for our users; non-negotiable (and this is needed for docker!)
*   Even if we could, we're still standardized on Windows 8.1
*   We want to ensure zero exposure to the baseline docker images; e.g. a single source of environment truth

Needless to say, your reasons may not mirror these. And that's okay. This may be an entirely irrevelvant configuration in your world, but - for me - this is useful. Let's look at a little chunk of a proof of concept having python/Flask calling the docker daemon API First up, define the baseline necessary json data structure for a container and some data for the docker host:

```python
from flask import Flask
import requests
import json

server = "DOCKER_HOST_IP_ADDRESS"
port = "2375"

containerTemplate = {
    "Image": 'ipy/iis',
    "ExposedPorts":
        {"80/tcp": None},
    "HostConfig":
        {"PublishAllPorts": True},
    "Cmd": [],
    "Env": []
}
```

Pretty simple to start with. IF you need documentation on a docker container creation and the possible values in the API, here you go: [API Docs](https://docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#/create-a-container) Right now, we're not defining much at all. Let's make a few notes though:

*   ```"Image": 'ipy/iis'```  - this is probably going to be an image you have created yourself; (perhaps I'll do a writeup on setting up an IIS docker container that pulls the live code on run, rather than build a little later!)
*   ```"ExposedPorts"``` - this is just one (of a number of) ways to setup exposed ports; in our scenario we're going to dynamically assign external ports and we'll hve our API get that back to the caller.
*   ```"Env": []```  - gives us an ability to setup environment variables. This is handy as we can now use the API to perform specific actions in the container on run; based on Environment Vaariables.

Next up; I feel it worth mentioning that when in a docker commandline and you run a "docker run img/imagename" its actually calling a create action on the API, followed by a run action. This needs to be done distinctly when using the API. Alrighty, now let's look at a few Flask routes that gets some stuff done.

```python
@app.route('/create', methods=['POST'])
def create():
    containerRequest = requests.post(("http://{0}:{1}/containers/create".format(server, port)), json=containerTemplate)
    data = json.loads(containerRequest.text)
    return data['Id']

@app.route('/createForTeam/<team>', methods=['POST'])
def createForTeam(team):
    #containerTest["Cmd"].append("powershell C:/scripts/pullWebSite.ps1 -Team {0}".format(team))
    containerTest["Env"].append("mobTeam={0}".format(team))
    containerRequest = requests.post(("http://{0}:{1}/containers/create".format(server, port)), json=containerTest)
    data = json.loads(containerRequest.text)
    return data['Id']

@app.route('/run/<containerId>', methods=['POST'])
def run(containerId):
    runningContainer = requests.post("http://{0}:{1}/containers/{2}/start".format(server, port, containerId))
    return "Running"

@app.route('/ports/<containerId>')
def ports(containerId):
    info = requests.get("http://{0}:{1}/containers/{2}/json".format(server, port, containerId))
    containerInfo = json.loads(info.text)
    cnPort = containerInfo["NetworkSettings"]["Ports"]["80/tcp"][0]["HostPort"]
    return "{0}".format(cnPort)
```

And now; let's look at how we'd call this from PowerShell.

```powershell
$containerId = Invoke-RestMethod -method Post -uri http://FLASK_APP_IP:5000/create
$state = Invoke-RestMethod -method post -uri http://FLASK_APP_IP:5000/run/$($containerId)
$ports = Invoke-RestMethod -method get -Uri http://FLASK_APP_IP:5000/ports/$($containerId)

#or

$containerId = Invoke-RestMethod -method Post -uri http://FLASK_APP_IP:5000/createForTeam/teamNameVariable
$state = Invoke-RestMethod -method post -uri http://FLASK_APP_IP:5000/run/$($containerId)
$ports = Invoke-RestMethod -method get -Uri http://FLASK_APP_IP:5000/ports/$($containerId)

#Oh, and just a simple quick way to hit the webpage:
Start-Process "chrome.exe" "DOCKER_HOST_IP_ADDR:$($ports)"
```

The second call, will hit the createForTeam route, which will take a value and insert an environment variable in the newly created container (which again, we can have the baseline container perform unique actions based on this value on run, rather than build!) All said, this is just an opening into really giving yourself a simplified, but supercharged API for docker management. We'll take this a step further in the next blog post.