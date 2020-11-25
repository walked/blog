---
title: "Azure RM - CustomData Injection"
date: 2016-12-29T12:32:30-05:00
draft: false
tags: ["windows", "powershell", "azure"]
---


In building out some Azure templates recently - once again - documentation turned out to be a bit thin. The goal was to inject custom data into a VM with data from the output of a resource created in the same deployment 


(I'll cover Azure RM template outputs in a different blog post). 

In my research, I came across [this article from Microsoft.](https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-linux-classic-inject-custom-data) Unfortunately, to quote the article: _This article covers using the Classic deployment model._ Darn it. I want to get some CustomData into my VM. Fear not, it's possible. Turns out; this can be done in the AzureRM template, and it works largely the same as the Azure Classic CustomData does. First off; the JSON syntax for you:

```json
        "osProfile": {
          "computerName": "[variables('vmName')]",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]",
          "customData": "[base64('testingInfo')]"
        }

```

The important piece to note is the base64 encoding function; this is critical as Azure will only accept base64 encoded values for CustomData. It will spit out a fun error if you try to skip that. 

**Now where's my data?** ```C:\AzureData\CustomData.bin<``` You have a few ways to access it:

```powershell
PS C:\AzureData\> $data = Get-Content C:\AzureData\CustomData.bin
PS C:\AzureData\> Write-Host $data
testingInfo
```

Or; you can simply ```Right Click -> Open With -> Notepad.exe``` 
_Obviously not quite as helpful for manual provisioning._ 

This is surprisingly frustrating in terms of documentation compared to what's available for AWS (see: userdata), but in the end it can largely successfully serve the same purpose. Hopefully Microsoft will get their AzureRM documentation pulled together :)