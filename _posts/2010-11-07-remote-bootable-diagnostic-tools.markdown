---
title: Remote Bootable Diagnostic Tools


date: 2012-03-24 18:36

author: Josh

category: Articles

tags: PXE, Tools

permalink: remote-bootable-diagnostic-tools
---

I have just opened up a preview of my new remote bootable Tech Tools
package.  To use it, simply download a \<1MB file, burn it to a CD or
flash drive (or even floppy eventually), and boot your computer from it.
 Your computer will download a few files from an Amazon S3 bucket, and
give you a menu of bootable diagnostic apps.  Currently, we
have [Memtest86+](http://memtest.org/), most manufacturer’s HDD
Diagnostic tools, [Darik’s Boot and Nuke](http://dban.org/), [Offline NT
Password & Registry Editor](http://pogostick.net/~pnh/ntpasswd/),
and [Ubuntu](http://ubuntu.com/) Lucid Lynx installers for all editions.
 More tools are on the way, listed on the Wiki.  I intend to keep all
these tools and more for free, possibly with some corporate sponsorship.
 Once the base gets finished, I’m going to try and build a Django/Python
app to allow you to upload your own tools and menus to run.  To test it,
download at <http://bit.ly/cobratech>.
