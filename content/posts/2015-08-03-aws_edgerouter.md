---
title: "Connecting Ubiquiti EdgeRouter to AWS VPC"
date: 2015-08-03T12:32:30-05:00
draft: false
tags: ["networking", "aws", "vpn"]
---



I'm of the mind that many small businesses can benefit greatly from extending their IT systems into the cloud. AWS is price effective (especially on a 1 year reserved term), and offers built-in VPN connectivity options via their VPC. It's really a quite powerful environment to work with.


For example, you could run a t2.small Windows Server 2012 instance for around $25 a month + VPN costs of around $35/month. $60 for a fully hosted domain controller including Windows Server licensing costs and Amazon's EC2 ability-to-scale/redundancy is pretty good.

Unfortunately for the small business/organization, many of the routers that will properly support the IPSec Site-to-Site tunnel can be burdensome in price. Enter the EdgeRouter Lite. 3 ports, based on VyOS 6.3, and $100 for 3xGigE ports. Really solid package. The downside? Documentation is a bit lacking. Ubiquiti sources a lot of documentation to the user community, and what they do publish can often not quite line up with your scenario. VyOS documentation is out there, but not always consistent.

So let's walk through an EdgeRouter to Amazon AWS VPC Configuration.


**Amazon Side:** _(I'm going to keep this simple and fast as the documentation on the Amazon side is pretty good)_

1. Ensure you have a VPC created and assigned a CIDR subnet. Mine is 172.31.0.0/16 
2. Create a Customer Gateway. This is a simple one-page configuration. Name it what you want, set the routing to static, and for the IP address use the _external IP for your router_. 
3. Create a Virtual Private Gateway. This is literally just creating a virtual router and takes no configuration. 
4. Create a VPN connection. Name it what you want, set the Virtual Private Gateway to the gateway created in step 3, use the existing customer gateway from step 2, and select "static" for the routing option. For IP prefixes, specify your own local subnet (e.g. 192.168.1.0/24) 
5. Right click on the created VPN connection and select "Download Configuration". Select the "Generic" vendor and download the text file. You have everything you need to connect. 

**Ubiquiti Side:** _(This is all via command line, sorry!)_

1. SSH to your router, PuTTY is going to be much easier to work with than the WebCLI due to copy/paste.
2. First, we need to create a Virtual Tunnel

```
configure
set interfaces vti vti0 address <AWS_PEER_IP-PROBABLY 169.X.X.X> 
## This IP address is listed in the Amazon Config as "Inside IP Addresses" - use the "customer gateway" here
set interfaces vti vti0 mtu 1436
```

3. Next, as a vti is a routable interface, we'll setup static routing for AWS

```
set protocols static interface-route <AWS_VPC SUBNET/MASK> next-hop-interface vti0 
#NOTE: This subnet is your VPC internal subnet. Mine is 172.31.0.0/16
``` 

4. Let's create the IPSec IKE Group and ESP Groups:

```
set vpn ipsec ike-group ikeGroup1
set vpn ipsec ike-group ikeGroup1 lifetime 28800
set vpn ipsec ike-group ikeGroup1 proposal 1 encryption aes128
set vpn ipsec ike-group ikeGroup1 proposal 1 hash sha1 
set vpn ipsec ike-group ikeGroup1 proposal 1 dh-group 2 
```

**Dead-Peer-Detection is SUPER important with AWS.**
**They will drop your connection like a hot potato if this isnt enabled and no traffic is passing.**

```
set vpn ipsec ike-group ikeGroup1 dead-peer-detection action 'restart'
set vpn ipsec ike-group ikeGroup1 dead-peer-detection interval '10'
set vpn ipsec ike-group ikeGroup1 dead-peer-detection timeout '30'    
set vpn ipsec esp-group espGroup1
set vpn ipsec esp-group espGroup1 compression disable
set vpn ipsec esp-group espGroup1 mode tunnel
set vpn ipsec esp-group espGroup1 pfs enable
set vpn ipsec esp-group espGroup1 lifetime 3600
set vpn ipsec esp-group espGroup1 proposal 1 encryption aes128
set vpn ipsec esp-group espGroup1 proposal 1 hash sha1
```

5. Now, we need to setup the IPSec Peer:

```
set vpn ipsec site-to-site peer <AWS PEER IP>
set vpn ipsec site-to-site peer <AWS PEER IP> authentication id <YOUR_EXTERNAL IP>
set vpn ipsec site-to-site peer <AWS PEER IP> authentication mode pre-shared-secret
set vpn ipsec site-to-site peer <AWS PEER IP> authentication pre-shared-secret <PRE_SHARED_SECRET>
set vpn ipsec site-to-site peer <AWS PEER IP> connection-type initiate
set vpn ipsec site-to-site peer <AWS PEER IP> default-esp-group espGroup1
set vpn ipsec site-to-site peer <AWS PEER IP> description "tunnel1"
set vpn ipsec site-to-site peer <AWS PEER IP> ike-group ikeGroup1
set vpn ipsec site-to-site peer <AWS PEER IP> local-address <YOUR_EXTERNAL_IP>
```

Note: The AWS Peer IP will be listed in the Amazon config file as "Virtual Private Gateway"

6. Final steps. We need to bind the peer to the vti and esp-group and finally mark our external interface as the ipsec-interface

```
set vpn ipsec site-to-site peer <AWS PEER IP> vti bind vti0
set vpn ipsec site-to-site peer <AWS PEER IP> vti esp-group espGroup1
set vpn ipsec ipsec-interfaces interface 'eth1' #Assuming your external interface is Eth1   
```

7. Verify connectivity!

```
user@router:~$ show vpn ipsec sa
Peer ID / IP                            Local ID / IP
------------                            -------------
X.X.X.X                                 Y.Y.Y.Y    
Description: tunne
Tunnel  State  Bytes Out/In   Encrypt  Hash  NAT-T  A-Time  L-Time  Proto
------  -----  -------------  -------  ----  -----  ------  ------  -----
vti     up     0.0/0.0        aes128   sha1  no     2967    3600    all
```


**Full Configuration Command List:**

    configure
    set interfaces vti vti0 address <AWS_PEER_IP-PROBABLY 169.X.X.X>
    set interfaces vti vti0 mtu 1436
    set protocols static interface-route <AWS_VPC SUBNET/MASK> next-hop-interface vti0
    
    set vpn ipsec ike-group ikeGroup1
    set vpn ipsec ike-group ikeGroup1 lifetime 28800
    set vpn ipsec ike-group ikeGroup1 proposal 1 encryption aes128
    set vpn ipsec ike-group ikeGroup1 proposal 1 hash sha1 
    set vpn ipsec ike-group ikeGroup1 proposal 1 dh-group 2
    set vpn ipsec ike-group ikeGroup1 dead-peer-detection action 'restart'
    set vpn ipsec ike-group ikeGroup1 dead-peer-detection interval '10'
    set vpn ipsec ike-group ikeGroup1 dead-peer-detection timeout '30'
    
    set vpn ipsec esp-group espGroup1
    set vpn ipsec esp-group espGroup1 compression disable
    set vpn ipsec esp-group espGroup1 mode tunnel
    set vpn ipsec esp-group espGroup1 pfs enable
    set vpn ipsec esp-group espGroup1 lifetime 3600
    set vpn ipsec esp-group espGroup1 proposal 1 encryption aes128
    set vpn ipsec esp-group espGroup1 proposal 1 hash sha1
    
    set vpn ipsec site-to-site peer <AWS PEER IP>
    set vpn ipsec site-to-site peer <AWS PEER IP> authentication id <YOUR_EXTERNAL_IP>
    set vpn ipsec site-to-site peer <AWS PEER IP> authentication mode pre-shared-secret
    set vpn ipsec site-to-site peer <AWS PEER IP> authentication pre-shared-secret <PRE_SHARED_SECRET>
    set vpn ipsec site-to-site peer <AWS PEER IP> connection-type initiate
    set vpn ipsec site-to-site peer <AWS PEER IP> default-esp-group espGroup1
    set vpn ipsec site-to-site peer <AWS PEER IP> description "tunnel1"
    set vpn ipsec site-to-site peer <AWS PEER IP> ike-group ikeGroup1
    set vpn ipsec site-to-site peer <AWS PEER IP> local-address <YOUR_EXTERNAL_IP>
    
    set vpn ipsec site-to-site peer <AWS PEER IP> vti bind vti0
    set vpn ipsec site-to-site peer <AWS PEER IP> vti esp-group espGroup1
    set vpn ipsec ipsec-interfaces interface 'eth1' #Assuming your external interface is Eth1

If you have more questions or need help, feel free to reach out!