---
title: Hardware Deployment with vPro

layout: default
category: Articles

tags: vPro Hardware

permalink: hardware-deployment
---

Every year at my job we deploy new machines, between 100 and 200. Every machine carries a 4+ year warranty, and they are staggered so we swap out about 1/4 every year. Each summer, we clean all the machines, deploy new ones, and remove old ones for testing and sale. In my first year, this accounted for a good portion of the summer. Thanks to the new deployment system my department designed, we cut that time down to about 2 weeks.

First, we deployed Intel vPro to every machine. This gives us the ability to remotely control every machine, remotely reboot (even when not in an OS or powered off), and deploy standard BIOS images either through Microsoft SCCM or via Serial Over LAN. The BIOS part saves some time during initial setup, and allows us to change BIOS settings if one department decides they need a new setting changed on all ~450 machines (like enabling VT). The remote rebooting makes the initial imaging much easier as well.

Next, we built a DBAN (Derrick's Boot and Nuke, a utility for wiping hard drives to DoD standards) TFTP server. Every machine is set to grab PXE instructions on every reboot (nightly for Windows machines, on command for Linux machines). We have a web interface that controls which server it grabs the PXE menu from and which file it boots from. I did a batch update, pointed them at our server, and modified the DBAN options to "autonuke" without user intervention. Now, we set every machine that will be replaced to boot from DBAN the morning before they are replaced (or manually tell them to in Linux's case), and by the time we get in, the machines are wiped and ready to be sold. We simply remove them, clean them on the way out, reset their BIOSs, and put in the new machine. It saves a lot of hassle with no extra downtime.

These simple changes saved us a lot of man hours, while giving us extra functionality. Win-win.