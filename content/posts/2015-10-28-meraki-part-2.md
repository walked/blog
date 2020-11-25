---
title: "Meraki VPN Deployment Part 2"
date: 2015-10-28T12:32:30-05:00
draft: false
tags: ["networking", "vpn"]
---

Well, humble pie time. But this is good stuff here. Turns out I missed one little teeny tiny flag in the Add-VpnConnection cmdlet. Oops. But this is good! 

<!--more-->

With PowerShell 4.0 you can deploy a fully functional Meraki VPN Client profile. _AND_ you can setup split tunneling. Granted; you cant push all the VPN subnets from the Meraki side; but this works and works pretty well. _Meraki guys - __take note because this is way better than your documentation._ So, first and foremost, let's see about JUST adding a VPN Profile that's compatible with Meraki gear in a single line:


```powershell 
Add-VpnConnection -L2tpPsk 'KEY' -name 'ConnectionName' -ServerAddress 'Endpoint IP or Host' -AllUserConnection -AuthenticationMethod Pap -TunnelType L2tp -Force
```

Cool. Easy. It was the **L2tpPsk** parameter that I'd missed. Darn it. Now lets talk about split tunneling. So, per the [Meraki documentation:](https://documentation.meraki.com/MX-Z/Client_VPN/Configuring_Split-tunnel_Client_VPN) 

Let's see here:

*   Can only be set when the VPN connection is up and running (ugh, this just about kills my deployment dreams)
*   Requires the ifIndex; which is only possible to pull when the VPN tunnel is up (Get-NetIPAdapter wont pull virtual disconnected adapters. Sigh)
*   ugh

Well, lets see here.. what else can we get done in PowerShell 4.0+? 

Enter: **Add-VpnConnectionRoute **
[https://technet.microsoft.com/en-us/%5Clibrary/dn262649(v=wps.630).aspx](https://technet.microsoft.com/en-us/%5Clibrary/dn262649(v=wps.630).aspx) 

Hm; well that's something. But how does this compare to the route commands in the Meraki documentation? I wonder...

![route]({{site.url}}/assets/images/meraki/route.png) 
What? So using the Add-VpnConnectionRoute does _not_ add the route to the route table. 

That's strange. I wonder what happens when I connect to the VPN connection.. 
![routes]({{site.url}}/assets/images/meraki/routes.png) 
YES! But... it's not listed as a persistent route. Does it survive a reboot? (Hint: Yes it does). So thats a whole lot of writing for basically two lines of PowerShell. I'm putting together a more-full script that'll parse an XML file full of subnets and VPN paramters, which will be posted here once it's firmed up! Until then, to deploy Meraki with split-tunneling via PowerShell 4.0+

```powershell 
Add-VpnConnection -L2tpPsk 'KEY' -name 'ConnectionName' -ServerAddress 'Endpoint IP or Host' -AllUserConnection -AuthenticationMethod Pap -TunnelType L2tp -SplitTunneling -Force
Add-VpnConnectionRoute -AllUserConnection -ConnectionName 'ConnectionName' -DestinationPrefix 10.1.1.0/24</pre>
```