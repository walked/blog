---
title: "Pushing Data to Dashing.io from PowerShell"
date: 2015-08-20T12:32:30-05:00
draft: false
tags: ["powershell"]
---
Dashing.io; super cool, and pretty fly. Easy enough to configure a quick and small linux VM to host it. But how about pushing data to it, from Powershell? The key is Invoke-RestMethod- Let's look at two of the simple built in widgets:
<!--more-->

```html
<li data-row="1" data-col="1" data-sizex="1" data-sizey="2">
      <div data-id="buzzwords" data-view="List" data-unordered="true" data-title="Buzzwords" data-moreinfo="# of times said around the office"></div>
    </li>

    <li data-row="1" data-col="1" data-sizex="1" data-sizey="1">
      <div data-id="valuation" data-view="Number" data-title="Current Valuation" data-moreinfo="In billions" data-prefix="$"></div>
    </li>
```

How would we push to these, assuming there's no Dashing.io jobs running? 
**For the Number Value (valuation):**

```powershell
$uri = "http://hostname.domain.tld:3030/widgets/valuation"
$body = @{
    auth_token = "YOUR_AUTH_TOKEN";
    current=[int]$value
}

Invoke-RestMethod -Method Post -Uri "$uri" -Body (ConvertTo-Json $body)</pre>
```

**For the List Value** (buzzwords): _A bit more complicated; but doable_

```powershell 
$auth_token = "YOUR_AUTH_TOKEN"
$dashing = "http://hostname.domain.tld:3030/widgets/buzzwords"

$items = @()
$items += @{label = "item 1"; value = "value"}
$items += @{label = "item 2"; value = "value2"}

$props = [ordered]@{
auth_token = "$auth_token"
items = $items
}

$body = New-Object -TypeName PSOBject -Property $props

ConvertTo-Json -InputObject $body

Invoke-RestMethod -Uri "$dashing" -Method Post -Body (ConvertTo-Json $body)</pre>
```

Pretty quick update; keeping it vague as implementation gets pretty environment specific pretty quickly! � But, one other note - if you wanted to test a simple true/false for a server being alive:

```powershell
Test-Connection -ComputerName hostname -Quiet
#This will return a true/false value! Awesome for quick testing.</pre>
```

Now go make some more dashboards.