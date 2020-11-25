---
title: "Azure RM - JsonADDomainExtension - Error 1332"
date: 2016-12-23T12:32:30-05:00
draft: false
tags: ["windows", "powershell", "azure"]
---

I've been working with Azure RM Templating quite a lot lately. It's actually pretty staggering in how versatile it is, but some of the documentation is a bit less than awesome. 

<!--more-->

First off, probably the best "documentation" resource I've found is the [Azure Quickstart Template repo.](https://github.com/Azure/azure-quickstart-templates) It's not formal documentation, but it's well organized and covers a whole lot of realistic scenarios. That said, the Domain Extension has a minor quirk that's worth noting; in my experience - it's identified by:

```json
{
    "code": "ComponentStatus/JoinDomainException for Option 3 meaning 'User Specified'/failed/1",
    "displayStatus": "Provisioning failed",
    "level": "Error",
    "message": "ERROR - Failed to join domain='MyAd.MyDomain', ou='CN=Computers,DC=MyAd,DC=MyDomain', user='USERNAME', option='NetSetupJoinDomain, NetSetupAcctCreate' (#3 meaning 'User Specified'). Error code 2",
    "time": null
}

{
    "code": "ComponentStatus/JoinDomainException for Option 1 meaning 'User Specified without NetSetupAcctCreate'/failed/1",
    "displayStatus": "Provisioning failed",
    "level": "Error",
    "message": "ERROR - Failed to join domain='MyAd.MyDomain', ou='CN=Computers,DC=MyAd,DC=MyDomain', user='USERNAME', option='NetSetupJoinDomain' (#1 meaning 'User Specified without NetSetupAcctCreate'). Error code 1332",
    "time": null
}
```

_The telling items being error 2 followed by 1332._ 

There's not much documentation in the way of this one out there. One other worthy note; the logs for the AD Join Domain extension can be found on the provisioned VM in the following directory: ```C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.JsonADDomainExtension\1.3\ADDomainExtension.txt```

As it turns out, this is as simple as this Azure Extension cannot join computers to the built-in Computers container within AD. It must be specified to join and place them in a separate OU path. Perhaps the fact that the parameter is "**OU**path" in the extension should have given it away; but nonetheless - you cannot join to ```CN=Computers,DC=MyAd,DC=MyDomain``` (Normally I'd have a staging OU configured, but I was testing this is a hastily deployed AD testing environment with a minimum of configuration. Once the Staging OU is provisioned and operational, all of this worked perfectly). The extension code for an RM template is as follows:

```json
{
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "[concat(parameters('virtualMachineName'),'/joindomain')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', parameters('virtualMachineName'))]"
      ],
      "properties": {
        "publisher": "Microsoft.Compute",
        "type": "JsonADDomainExtension",
        "typeHandlerVersion": "1.3",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "Name": "[parameters('domainName')]",
          "User": "[parameters('userName')]",
          "Restart": "true",
          "Options": "3",
          "OUPath": "OU=Staging,DC=MyAd,DC=MyDomain"
        },
        "protectedsettings": {
          "Password": "[parameters('adminPassword')]"
        }
      }
    }
```