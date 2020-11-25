---
title: "Docker - Multiplatform Swarm and Bugs"
date: 2016-10-21T12:32:30-05:00
draft: false
tags: ["docker", "windows"]
---
Docker is so cool; and supporting Windows and Linux alike is even cooler. High availability is a thing though, so I was delighted to find out I can create a swarm in Linux; join Windows Server 2016, and have it managed. Awesome! But...
<!--more-->


Easy enough to do too: 

**Linux side:** _(manager node)_

```
$ sudo docker swarm init --advertise-addr 192.168.1.229
Swarm initialized: current node (3i4tob1jc5xlpo09oak97u42n) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-2ya724dqobyyay67qo2rb9h76xzcof8d54ev5hqpfyzfnam5k4-7kk0e96myxm3ermztepjb6ijb \
    192.168.1.229:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

**Windows side:** _(worker node)_

```
PS C:\> docker swarm join --token SWMTKN-1-2ya724dqobyyay67qo2rb9h76xzcof8d54ev5hqpfyzfnam5k4-7kk0e96myxm3ermztepjb6ijb 192.168.1.229:2377
This node joined a swarm as a worker.
```

Super cool! Now lets deploy a service in swarm mode! 

**Manager:**

```
~$ sudo docker service create --name test microsoft/iis
aurckil3iacf57f0smoofn4io
```

And then; we have on the **worker node** _(windows side)_

```
PS C:\> docker ps
CONTAINER ID        IMAGE                  COMMAND                   CREATED             STATUS                  PORTS 
              NAMES
45b102072d5e        microsoft/iis:latest   "C:\\ServiceMonitor..."   7 seconds ago       Up Less than a second         
              test.1.6wtl63hq2trf1vhetcnhku2xm
```
  
Great. But then it gets kinda bad. 

Because if we try to publish a port; it fails. It doesnt fail gracefully, it doesnt provide feedback, it just does _nothing_ Let's fire up a service, and expose port 80 in the swarm

```
~$ sudo docker service create --name noBueno --publish 80 microsoft/iis
0r3sgr8k6ioi9480fjxmb8y9u
```

So far so good. Let's check out the Windows host:

```
PS C:\> docker ps -a
CONTAINER ID        IMAGE                  COMMAND                   CREATED             STATUS              PORTS     
          NAMES
45b102072d5e        microsoft/iis:latest   "C:\\ServiceMonitor..."   3 minutes ago       Up 3 minutes                  
          test.1.6wtl63hq2trf1vhetcnhku2xm
```

Huh; weird. No new containers running. Let's look at the management node's service status:

```
~$ sudo docker service ls
ID            NAME     REPLICAS  IMAGE          COMMAND
0r3sgr8k6ioi  noBueno  0/1       microsoft/iis  
aurckil3iacf  test     1/1       microsoft/iis
```

No replicas. Never even shows up on the Windows server. And to add to this, to prove it's a bug, let's publish a port retroactively on the already running container:

```
~$ sudo docker service update --publish-add 80 test
```

And hop over to our Windows host...

```
PS C:\> docker ps -a
CONTAINER ID        IMAGE                  COMMAND                   CREATED             STATUS                        
 PORTS               NAMES
45b102072d5e        microsoft/iis:latest   "C:\\ServiceMonitor..."   6 minutes ago       Exited (1067) 35 seconds ago  
                     test.1.6wtl63hq2trf1vhetcnhku2xm
```

Nooooooooooooo! 

Bug report has been submitted: <https://github.com/docker/docker/issues/27612>   Let's hope this gets some action in the near(ish) future, because as far as I can tell - a Windows node cannot be the docker swarm manager, and multi-platform swarms certainly arent prime-time yet.