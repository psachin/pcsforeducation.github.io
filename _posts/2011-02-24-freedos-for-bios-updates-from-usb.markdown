---
title: FreeDOS for BIOS updates from USB


date: 2011-02-24 15:23

author: Josh

category: Articles

tags: Tips

permalink: freedos-for-bios-updates-from-usb
---

We've all been there. Or you are there right now if you're reading this.
Some manufacturer still believes it is 1995, and we all have a floppy
drive laying around. I mean, I do have a stack of floppy drives, but
that is not the point! I tried like 10 tutorials today trying to get a
simple install of FreeDOS onto a flash drive with the BIOS update I
needed. This is the only one that worked for me!

<http://manual.aptosid.com/en/bios-freedos-en.htm>

However, I'm going to modify and restate what they said for my own
notes, and maybe for yours.

First, you need to know which drive letter your USB stick is. Head over
to /dev before putting in your USB stick.

{% highlight bash %}
cd /dev
ls
{% endhighlight %}

We are looking specifically for things starting with "sd". You should
definitely have at least "sda". That is your first SATA/SCSI hard drive.
Anything like "sda1" is a partition. We are only concerned with full
devices for now.

Plug your drive in, and run "ls" again.

Now you should have a new "sd" entry. Remember this! We'll assume it is
/dev/sdb for this tutorial.

We need to reformat the drive, so if you have anything on it, back it
up! This assumes you just have one partition, and it is not formatted as
MSDOS right now. If you have issues, view the original tutorial for a
more indepth guide to repartitioning your flash drive.

{% highlight bash %}
sudo mkfs.msdos /dev/sdb1
{% endhighlight %}

To install, we are going to use the virtualization tool qemu. Install
like so:

{% highlight bash %}
sudo apt-get -y install qemu
{% endhighlight %}

And we need the FreeDOS CD image to install. Grab it from here:

[FreeDOS Base
CD](http://www.ibiblio.org/pub/micro/pc-stuff/freedos/files/distributions/1.0/fdbasecd.iso)

Or (if the above link breaks)

[FreeDOS Download Site](http://www.freedos.org/freedos/files/)

Now, let's boot FreeDOS!

{% highlight bash %}
sudo qemu -hda /dev/sdb -cdrom fdbasecd.iso -boot d
{% endhighlight %}

At that point, hit 1 to start installation, and follow the prompts to
fully install it. Check the tutorial for a more indepth walkthrough.

Finally, download whatever files you need to flash the BIOS (or whatever
else you want to do), and copy them to the root of the drive. You should
be able to boot and follow the manufacturer's instructions to flash the
BIOS! Awesome!
