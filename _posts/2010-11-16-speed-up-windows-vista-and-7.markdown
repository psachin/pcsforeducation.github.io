---
title: Speed Up Windows Vista and 7


date: 2010-11-16 20:54

author: Josh

category: Articles

tags: Windows

permalink: speed-up-windows-vista-and-7
---

When your computer starts up, a defined set of “services”, or mini
components of Windows, are systematically started, and some keep running
the entire time your computer is running.  Each running service takes up
some amount of memory and processing power, which in turn, slows down
your computer.  Not cool! However, since Windows is used by over 90% of
computer users, the defaults need to work for **everyone**, from
business users, to beginners, to power users at home.  For the average
user, a lot of these can be disabled, making your computer much faster! 
The site BlackViper.com provides a list of all services, and tells you
which ones you can disable, based on how far you want to push your
computer.  At the end of the article, I provide the links to two
registry tweaks that will make this tedious task much quicker, if you
are running Vista 32-bit edition. (If you have less than 3GB of RAM, you
are probably running 32-bit.) Tweaks for XP and for Windows 7 will be
added later, along with Vista 64-bit.

Let’s get started! First of all open the run command, by either hitting
[Windows Key] + [R] or going to the start menu and clicking “run”. Type
“services.msc” and hit “OK”.  You can manually go through each service
and right-click, then go to “properties”, followed by changing the drop
down box to “Automatically”, “Manual”, or “Disabled”. Let’s try one:

​1. Find “Distributed Link Tracking Client” in the list and right-click
on it, then click “properties”. 2.Click the drop-down box next to
“Startup Type:” and change it to “Disabled”. 3. Click “OK” to continue.
4. Restart your computer for the change to take effect.

It is that easy! However, for the Tweaked edition (the less safe of the
two, but faster), you would have to change a whopping 52 services! That
would take forever. To get the full list of what to change and what not
to change, check Black Viper’s site, and click on the appropriate links
(Vista, XP, 7, 64-bit, or 32-bit).   However, the site doesn't have
pre-made "Tweaked" registry edits, so I modified one (should work for
Vista/7 both 32 and 64 bit).

[BlackViper Vista/7
Tweaked](http://servercobra.com/downloads/BlackViperVista7tweaked.zip)

Download the file, open it up, and run the registry fix inside.  If you
have any issues, please shout them out in the comments! (It does say
there is an error when I run it on my laptop, but still changes the keys
as far as I can tell.)
