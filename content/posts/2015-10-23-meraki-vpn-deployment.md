---
title: "Meraki VPN Deployment"
date: 2015-10-23T12:32:30-05:00
draft: false
tags: ["networking", "vpn"]
---

Grumble. We're in the process of replacing our infrastructure with Meraki products - in fact, we've been _extremely_ pleased with this so far. Except for that **one thing**. Deploying the Client VPN _sucks_.

<!--more-->

 Yeah. It sucks. Let's talk about the requirements of the Meraki VPN, and some of the methods out there for deploying Windows native VPN profiles. First off, the Meraki documentation on Client VPN settings: [https://docs.meraki.com/display/MX/Client+VPN+settings](https://docs.meraki.com/display/MX/Client+VPN+settings) Cool. What's worth noting:

*   Type of VPN Must be L2TP
*   Allow these protocols:
    *   Unencrypted password (PAP)
*   Advanced Settings:
    *   "Use a preshared key for authentication" is checked
    *   Preshared key is typed out

Pretty straightforward. Lets see about some deployment options now: 

**Failure 1: SCCM 2012 R2** 
SCCM cant do it; no way to set the Unecrypted PAP setting as far as I can tell, nor a Windows-native PSK. 

**Failure 2: GPO** 
GPO cant do it either; nowhere to set the PSK. 

**Failure 3: Copying the PBK file** 
As documented here: [http://www.bauer-power.net/2011/09/how-to-back-up-restore-or-deploy.html](http://www.bauer-power.net/2011/09/how-to-back-up-restore-or-deploy.html) Yet another failure - for you see; it'll keep ALL the options in hand, but wont retain the pre-shared key. 

**Failure 4: Straight Powershell** WMF 4.0 has the new Add-VpnConnection cmdlet. Awesome. Nope. Cant set the PSK. 
[![banging-head-on-wall-1-reduced](http://i-py.com/wp-content/uploads/2015/10/banging-head-on-wall-1-reduced-300x225.jpg)](http://i-py.com/wp-content/uploads/2015/10/banging-head-on-wall-1-reduced.jpg)   

Ok. So what to do? Time to get creative. We have a deployment ahead of us, and our entire userbase depends on this VPN profile. Enter: The dotRas SDK: [https://dotras.codeplex.com/](https://dotras.codeplex.com/) Awesome! I can just put together a quick exe and there we ...What? No way to set the Unencrypted PAP setting required by Meraki. ![](http://i-py.com/wp-content/uploads/2015/10/Dr.-Who.gif) 
Alright. Fine. Through some crafty work with dotRas, _and_ Powershell invocation - we can do this!

```csharp
                pb.Open();
                string vpnname = "Name for VPN";
                string pskey = "PreShared Key";
                string endpoint = "Destination IP or Hostname";

                RasEntry vpnentry = RasEntry.CreateVpnEntry(vpnname, endpoint, RasVpnStrategy.L2tpOnly, RasDevice.Create(vpnname, RasDeviceType.Vpn));
                vpnentry.Options.UsePreSharedKey = true;
                vpnentry.EncryptionType = RasEncryptionType.Optional;
                vpnentry.Options.RequirePap = true;
                pb.Entries.Add(vpnentry);
                vpnentry.UpdateCredentials(RasPreSharedKey.Client, pskey);

                PowerShell ps = PowerShell.Create();
                ps.AddScript("Set-VpnConnection -Name \"" + vpnname + "\" -AllUserConnection -AuthenticationMethod Pap");

                ps.Invoke();
```

So there you go. In the end, it's totally doable (even if it's not my favorite approach ever). This is just a snippet; and I've put together a full mini-application-thing that reads the connection settings from an XML file and puts it into network connections. At some point in the coming day or two I'll get around to uploading it to github and post it here. Developers: Please note - I'm not a developer. Ignore my iffy coding skills. Thank you! What's the downside? The PSK is stored in plain text. I'm not a huge fan of that; and may get around to encrypting it and updating the GitHub project accordingly; however it's not exactly a top priority for my purposes. Until then, I'll be holding a slight grudge against Meraki. 

EDIT: Alright; up on GitHub. **[https://github.com/walked/VpnProfile](https://github.com/walked/VpnProfile)**