---
title: "Cmder Lambda"
date: 2016-05-04T12:32:30-05:00
draft: false
tags: ["cli"]
---

So I have to say, I'm just enamored with [Cmder](http://cmder.net/) It's phenomenal. Elegant, clean, and just truly the best console application I've used. Throw it in a dropbox and you're _set. _SSH, git, grep... all my *nix aliases in one portable package? Sure thing - I'm sold. And it's pretty. Let's customize it a teeny, tiny bit. I wont re-write the featureset, so:

> #### Git and others
> 
> Oooh yes! If you decide to use the _slightly_ bigger [msysgit](https://msysgit.github.io/) version, you will have all Unix commands ready in PATH so that you can `git init` or `cat` instantly on every machine.
> 
> #### Total portability
> 
> Carry it with you on USB stick or in Cloud. So your settings, aliases and history can go anywhere you go. You will not see that ugly Windows prompt ever again.
> 
> #### With help of the best
> 
> Think about cmder more as a software package than a separate app. All the magic is happening through [Conemu](https://conemu.github.io/). With enhancements from [Clink](https://mridgers.github.io/clink/).

But man; that Lambda at the command prompt... just irks me.

So how do you fix it? There's two places to edit:

1.  ```<cmder_dir>\config\cmder.lua```  As you can see; I'm partial to the carat (> ) but you can use whatever you want.
2.  ```<cmder_dir>\vendor\profile.ps1``` This was a bit trickier to track down; but there it is. You may want to edit this so you know when you're in PowerShell; but its up to you.

Enjoy!