---
title: "Disabling Screen Blanking - Raspian Jesse"
date: 2016-10-07T12:32:30-05:00
draft: false
---

Another little tidbit I cam across in setting up a Raspia Raspberry Pi 3 (model b) for use in a kiosk setting (actually just an always on system monitor, but same idea). After X minutes, the screen blanking setting will come into play;

<!--more-->

 and the screen will blank after about 10min of no keyboard input. Even if there's an active program running. _Unacceptable!_ Easy fix, right? 



Just about every blog has the fix: https://www.bitpi.co/2015/02/14/prevent-raspberry-pi-from-sleeping/ https://www.raspberrypi.org/forums/viewtopic.php?f=66&t=18200 http://raspberrypi.stackexchange.com/questions/752/how-do-i-prevent-the-screen-from-going-blank Not so fast. So, all these blogs pretty much agree; it's super simple. Just make a few changes:

```
sudo nano /etc/kbd/config
```

And then

```
BLANK_TIME=0
POWERDOWN_TIME=0
```

Just add those changes; seems simply enough. Reboot and you're good to go. Right? Nope. It doenst work in Jesse; thanks to a bug between kbd and systemd. It took me forever, and ever, and ever to track down a discussion on this. But sure enough there's one brief little chat on it out there. https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=771161 Ah. Wait.

```Date: Thu, 27 Nov 2014 09:15:02 UTC
// seriously?!
```

Oh well; the workaround is there. And it works: Just modify /etc/init.d/kbd

```
sudo nano /etc/init.d/kbd
```

And then add this:

```
TERM=linux setterm > /dev/tty1 $setterm_args 
```

And then reboot again. Hooray, now I have a working Raspian console that doesnt die without active input.