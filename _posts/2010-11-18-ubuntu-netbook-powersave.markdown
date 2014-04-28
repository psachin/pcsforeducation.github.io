---
title: Ubuntu Netbook PowerSave


date: 2010-11-18 12:26

author: Josh

layout: default
category: Articles

tags: Ubuntu

permalink: ubuntu-netbook-powersave
---

The first thing I do when I reinstall Ubuntu on my netbook is set up
power saving stuff (after getting the proprietary wifi drivers over
ethernet.....grr..).  Here's a handy way to make sure you enable power
saving features every time you boot the computer. We are going to create
a file called S99powersave in the /etc/rc5.d/ directory, which is a set
of scripts that gets run when going into runlevel 5 (the default Ubuntu
runlevel when you boot up).   S99 denotes the order in which the scripts
are run.  The lower numbers get executed first.  Since we want to be the
least intrusive as possible, we are going to put it as one of the last
scripts to be run. This could also be put into the file /etc/rc.local,
with similar results.

{% highlight bash %}
sudo nano /etc/rc5.d/S99powersave
{% endhighlight %}

{% highlight bash %}
# Enable power save for Intel audio devices
echo 1 > /sys/module/snd_hda_intel/parameters/power_save

# Enable SATA link power management
echo min_power > /sys/class/scsi_host/host0/link_power_management_policy

# Turn bluetooth off (adds 2 hours of life to my netbook, up to 9 hours total)
/sbin/rfkill block bluetooth

# Increase Virtual Memory dirty writeback time
echo 1500 > /proc/sys/vm/dirty_writeback_centisecs

# Enable wireless power management
/sbin/iwconfig wlan0 power timeout 500ms
{% endhighlight %}

The way I found these commands was by installing and running powertop
for Ubuntu. It gives you an easy way to disable some stuff, and then the
command to make it happen. Please run it on your computer and add any of
the output powertop shows you, so we can have a more complete script!
