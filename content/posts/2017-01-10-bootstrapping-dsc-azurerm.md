---
title: "AzureRM - Bootstrapping DSC LCM"
date: 2017-01-10T12:32:30-05:00
draft: false
tags: ["windows", "powershell"]
---

So, a brief tidbit if you're not using Azure automation for your Azure VMs (we have a subset that are managed with a pull server that are not Azure automation controlled. So, at provisioning time we wanted to look at provisioning the VM with the LCM meta configuration, to avoid additional work by hand. It's pretty simple, but required a different approach between WMF 4.0 and WMF 5.0 (read: Server 2012 and Server 2016). Let's discuss:

<!--more-->

### **Server 2016** - 

As it turns out the PowerShell DSC Extension doesnt  support ```[DSCLocalConfigurationManager()]``` . Darn. So, the next best thing is a hyper-simple script running in the Custom Script Extension:

```powershell
[DSCLocalConfigurationManager()]
configuration PullClientConfigID
{
    Node localhost
    {
        Settings
        {
            RebootNodeIfNeeded   = $true

        }

    }
}

PullClientConfigID -OutputPath C:\Config

Set-DscLocalConfigurationManager -Path C:\Config
```

_Note: this LCM config left intentionally simple!_ And then we just set up the Azure RM template with a Custom Script Extension:

```json
{
              "name": "configLcm",
              "type": "extensions",
              "location": "[resourceGroup().location]",
              "apiVersion": "2015-06-15",
              "dependsOn": [
                  "[resourceId('Microsoft.Compute/virtualMachines', variables('vmName'))]"
              ],
              "tags": {
                  "displayName": "configLcm"
              },
              "properties": {
                  "publisher": "Microsoft.Compute",
                  "type": "CustomScriptExtension",
                  "typeHandlerVersion": "1.4",
                  "autoUpgradeMinorVersion": true,
                  "settings": {
                      "fileUris": [
                          "[concat(parameters('_artifactsLocation'), '/', variables('configLcmScriptFolder'), '/', variables('configLcmScriptFileName'), parameters('_artifactsLocationSasToken'))]"
                      ],
                      "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File ', variables('configLcmScriptFolder'), '/', variables('configLcmScriptFileName'))]"
                  }
              }
          }
```

And we're off to the races upon provisioning.

### Server 2012 R2

This is a situation where we can simply use the DSC extension in the Azure RM template. Simpler.

```powershell
Configuration Main
{

Param ( [string] $nodeName )

Import-DscResource -ModuleName PSDesiredStateConfiguration

Node $nodeName
  {
   LocalConfigurationManager
        {
            RebootNodeIfNeeded = $true
        }
  }
}
```

And then, the supporting template configuration:

```json
{
              "name": "Microsoft.Powershell.DSC",
              "type": "extensions",
              "location": "[resourceGroup().location]",
              "apiVersion": "2015-06-15",
              "dependsOn": [
                  "[resourceId('Microsoft.Compute/virtualMachines', variables('vmName'))]"
              ],
              "tags": {
                  "displayName": "configLcm"
              },
              "properties": {
                  "publisher": "Microsoft.Powershell",
                  "type": "DSC",
                  "typeHandlerVersion": "2.9",
                  "autoUpgradeMinorVersion": true,
                  "forceUpdateTag": "[parameters('configLcmUpdateTagVersion')]",
                  "settings": {
                      "configuration": {
                          "url": "[concat(parameters('_artifactsLocation'), '/', variables('configLcmArchiveFolder'), '/', variables('configLcmArchiveFileName'))]",
                          "script": "configLcm.ps1",
                          "function": "Main"
                      },
                      "configurationArguments": {
                          "nodeName": "[variables('vmName')]"
                      }
                  },
                  "protectedSettings": {
                      "configurationUrlSasToken": "[parameters('_artifactsLocationSasToken')]"
                  }
              }
          }
```

  Pretty straightforward. In a more realistic example (and what we're doing) you'd also configure your pull / report server and any other LCM metadata you want to setup.