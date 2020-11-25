---
title: "Cmder changing the lambda - update!"
date: 2017-01-18T12:32:30-05:00
draft: false
tags: ["cli"]
---

So previously discussed cmder before: [Cmder - Changing the Lambda]({{ site.baseurl }}{% post_url 2016-05-04-cmder-lambda %})

However, with the newest releace of Cmder there were some file structure changes that were made that need to be updated for fixing the lambda. 

<!--more-->

### For the regular cmd prompt:
 ```<cmder_dir>\vendor\clink.lua```  Line number 43 and 46.

```lua
    if env == nil then
        lambda = ">" -- line  43
    else
        lambda = "("..env..") >" -- line 46
    end
```


### For the PowerShell prompt:
 ```<cmder_dir>\vendor\profile.ps1``` Line number 168+. This was a bit trickier to track down; but there it is. You may want to edit this so you know when you're in PowerShell; but its up to you.

```powershell
[ScriptBlock]$Prompt = { #line 168
    $realLASTEXITCODE = $LASTEXITCODE
    $host.UI.RawUI.WindowTitle = Microsoft.PowerShell.Management\Split-Path $pwd.ProviderPath -Leaf
    PrePrompt | Microsoft.PowerShell.Utility\Write-Host -NoNewline
    CmderPrompt
    Microsoft.PowerShell.Utility\Write-Host " PS> " -NoNewLine -ForegroundColor "DarkGray" ### this is my edit
    PostPrompt | Microsoft.PowerShell.Utility\Write-Host -NoNewline
    $global:LASTEXITCODE = $realLASTEXITCODE
    return " "
}
```