---
title: Setup PXE Server


date: 2010-11-18 12:34

author: Josh

category: Articles

tags: PXE, Ubuntu

permalink: setup-pxe-server
---

**Overview** By popular demand, I broke the basic PXE Server bit into
it’s own section. This is mostly a rehash of what is available in the
Diskless Eucalyptus Cluster tutorial and Diskless <Folding@Home> Cluster
tutorial. After my new found love affair with Python, I should have a
Python script to automate this in the next few days, especially to go
along with my EasyClusterController project.

This tutorial will cover how to install and setup a basic PXE boot
server, using dnsmasq and tftpd-hpa on Ubuntu. This article assumes you
don’t have another DCHP server on your network (like a router). I will
update soon with a tutorial on segregating networks by using 2 network
cards on the server, thereby setting up 2 separate networks with a the
server acting as a router between the two.

**Platform(s)**: Ubuntu 8.04 – 10.04

I often find it is easier to understand a concept if you have a nifty
diagram. There’s a lot of stuff going on to get a PXE boot to work, and
if you know how it works, it will be much easier to diagnose if
something goes wrong, and to build on top of it. The following diagram
was built with Dia for Linux (in case you were wondering).

**PXE Boot Server for Home Servers** **Installation** First of all,
either install directly from the Ubuntu Server CD (or other CD, such as
desktop, though the Server CD will run fastest), selecting options as
you please. You can build this server on an already running system as
well.

First, let’s make sure everything is up to date:

{% highlight bash %}
sudo apt-get -y update && apt-get -y dist-upgrade
{% endhighlight %}

Let’s get the packages we need:

{% highlight bash %}
sudo apt-get install dnsmasq tftpd-hpa syslinux initramfs-tools
{% endhighlight %}

(Other guides will tell you to use dhcp3-server in place of dnsmasq, but
I find dnsmasq much simpler to use in a small network. If you’re going
for a large deployment with lots of customization, you should probably
use dhcp3-server. However, I cannot think of any time you would need the
added functionality of dhcp3-server in a home network. Also, tftpd-hpa
is not required. You can set up dnsmasq to be a tftp server, but I was
having troubles allowing clients to write to the tftp directory, so I
use tftpd-hpa to make sure there are no issues).

**Configure networking** Now we’re going to set up a static IP so all
the computers on the network know which one is the DHCP and boot server.

{% highlight bash %}
sudo nano /etc/network/interfaces
{% endhighlight %}

Change “iface eth0 inet dhcp” to:

{% highlight bash %}
iface eth0 inet static
address 192.168.1.10
netmask 255.255.255.0
network 192.168.1.0
broadcast 192.168.1.255
gateway 192.168.1.1
{% endhighlight %}

Remove dhcp3-client to let the update go into effect:

{% highlight bash %}
sudo apt-get -y remove dhcp3-client
{% endhighlight %}

Restarting network for all updates to go into effect:

{% highlight bash %}
sudo /etc/init.d/networking restart
{% endhighlight %}

**Set up dnsmasq**
:   Edit the dnsmasq configuration file:

{% highlight bash %}
sudo nano /etc/dnsmasq.conf
{% endhighlight %}

 

{% highlight bash %}
#Makes dnsmasq the authoritative dhcp server for the network (instead of any router), then tells what IP's it
#is allowed to hand out. Don't use with another router on the network (could be very bad!)

dhcp-authoritative
dhcp-range=192.168.1.21,192.168.1.99,6h
#Tells the server where to boot from. Replace cloudcontroller with your server's hostname
dhcp-boot=pxelinux.0,cloudcontroller,192.168.1.10
#Assigns certain MAC address a specific IP and hostname
dhcp-host=00:00:00:00:00:00,cloud-dl380,192.168.1.11
dhcp-host=00:00:00:00:00:00,cloud-rioworks,192.168.1.12
{% endhighlight %}

You can find the MAC address if you boot up each server, and tell it to
network boot. It will display the MAC/Hardware address. You can find the
MAC address if you boot up each server, and tell it to network boot. It
will display the MAC/Hardware address. Replace the 00:00:00:00:00:00:00
with the actual numbers of each server. This isn’t absolutely necessary,
but I find it is extremely helpful in keeping track of what is going on.

Let’s restart dnsmasq and make sure it works:

{% highlight bash %}
sudo /etc/init.d/dnsmasq restart
{% endhighlight %}

**Configure TFTP**

First, we need to point TFTP at the correct directory.

{% highlight bash %}
sudo nano /etc/default/tftpd-hpa
-----
# Change the line starting with TFTP_DIRECTORY= to
TFTP_DIRECTORY="/opt/tftp"
-----

# Now we restart TFTP to let changes take effect.
/etc/init.d/tftpd-hpa
{% endhighlight %}

Let’s prepare our server to serve requests over TFTP, and get the
required files into the TFTP directory, both the pxelinux.0 file that
talks to your network card so it can boot without any disk drives, and
the kernel you are currently using to boot your Ubuntu computer. The
last line uses a bit of command-line trickery to copy the kernel based
on your current version of Linux.

{% highlight bash %}
sudo mkdir -p /opt/tftp/pxelinux.cfg
sudo cp /usr/lib/syslinux/pxelinux.0 /opt/tftp
cp /boot/vmlinuz-$(uname -r) /opt/tftp
{% endhighlight %}

We need to edit the PXE configuration file so our clients can boot.

{% highlight bash %}
sudo nano /opt/tftp/pxelinux.cfg/default
{% endhighlight %}

 

{% highlight bash %}
#Update to be the file names to the files copied from /boot/vmlinuz
LABEL linux
KERNEL vmlinuz-2.6.31-14-generic-pae
APPEND root=/dev/nfs initrd=initrd.img-2.6.31-14-generic-pae nfsroot=192.168.2.2:/opt/nfsroot ip=dhcp rw
{% endhighlight %}

You can also configure a file per MAC address that will be booting, by
creating a file in the same directory by the name of ‘XX-XX-XX-XX-XX-XX’
without the quotes and replacing the X’s with the MAC address of the
computer to use that file.

Let me explain the configuration file a bit. The kernel will be the
kernel name that we copied out of the /boot directory. The APPEND line
is for diskless clients that mount an NFS shared directory. You will
have to change this to reflect what you are doing. Since this article is
meant to be a springboard for many different types of uses for PXE, I’ll
leave that to show you how it can be used, but other tutorials will
probably dictate exactly how this should be configured.

Finally, you’ll need a ramdisk image (the initrd.img from the config
file above). This is going to depend on what you’re booting and how
you’re connecting to the filesystem on the clients. The above is
configured for diskless boot, and the image is made on one of the
clients, so I’ll show you how to do that. Edit the initrd config file:

{% highlight bash %}
sudo nano /etc/initramfs-tools/initramfs.conf
{% endhighlight %}

Find the lines starting with MODULE and BOOT, and change them to the
following:

{% highlight bash %}
MODULE=netboot

BOOT=nfs
{% endhighlight %}

Now lets create a ramdisk.

{% highlight bash %}
sudo mkinitramfs -o /opt/tftp/initrd.img-\`uname -r\`
{% endhighlight %}

As vague as that is, you’re probably using this tutorial with another
tutorial, such as Fully Automated Installation server. Check the
Diskless Ubuntu tutorial or the Diskless Eucalyptus Cluster tutorial for
actual instructions on how to do this.

**Conclusion** That should get you a working boot server. In the next
tutorial I’ll show you how to prepare your cluster computers to boot
without any hard disks, building off this notion. Also, the Fully
Automated Installation server builds off this tutorial.
