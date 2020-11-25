---
title: "Initial Release of Route53 DynamicDNS Client"
date: 2017-11-02T12:32:30-05:00
draft: false
tags: ["windows", "terraform", "aws", "golang"]
---

There's many Dynamic DNS clients; some support Route53. All of them seem to have a bunch of dependencies or be platform specific. Go is really good for this, so why not cook something up really quick? 
 
<!--more-->

**GitHub:** <https://github.com/walked/route53ddns>

So why this release?

First and foremost: It's entirely possible something out there does what I want it to. However, the aim here was a few things:
- I want to compile to a single executable 
- I wanted cross-platform compatability
- I wanted to not be dependent upon the AWS CLI credential provider

Neat. This solved those items!

Setup:
- Extract the 7z into a directory; for me; `c:/tools/ddns/`
- Open a command prompt; navigate to the directory `cd c:/tools/ddns/`
- Run `route53ddns.exe --configure` 
- Give it the info

You can then run this by hand; or run via Windows Task Scheduler.

(Note if you run via task scheduled; it's really important to set the `Start In` property to the directory with the exe/toml file (e.g. `c:/tools/ddns/`))

