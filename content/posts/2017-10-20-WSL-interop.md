---
title: "WSL - Interop Tip"
date: 2017-01-29T12:32:30-05:00
draft: false
tags: ["windows" ]
---

**NOTE 11/25/2020:** This post is very, very out of date. This is all WSL1. WSL completely revamps and fixes this approach and makes this entire post obsolute. Do not follow the directions below in 2020+

Quickest post yet; but maybe it will help someone! Because the WSL allows you to call Windows binaries from the Linux system, there's a quick trick for paths that I've run into. 

<!--more-->

First; where is the filesystem of the linux system on your Windows host? Assuming the Fall Creators Update, installed via the store (Ubuntu in this case):

```
C:\Users\USERNAME\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_GUIDHERE\LocalState\rootfs
```

Cool.

However, let's say I have a file ```test.txt``` in my home directory that I want to edit using Visual Studio Code. Hmm.

I can type:
```code ./test.txt```
However, this will pass the argument to the Windows binary unmodified. Darn it. I need to get the path to it from a Windows frame of reference; not the Linux frame of reference..

The trick? Create a new environment variable in your bash profile that exports the full Windows path, which will then be expended by bash prior to passing to the Windows binary.

Like this:
![Shell]({{site.url}}/assets/images/wsl/path.gif)

```export WIN=C:\\Users\\USENAME\\AppData\\Local\\Packages\\CanonicalGroupLimited.UbuntuonWindows_GUID\\LocalState\\rootfs```
(for anyone who wants to copy and paste)

This will then open up code; able to save, and able to use it to edit WSL files. Pretty awesome capability.

