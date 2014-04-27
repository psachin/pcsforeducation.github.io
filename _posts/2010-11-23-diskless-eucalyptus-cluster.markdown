---
title: Diskless Eucalyptus Cluster

date: 2010-11-23 11:21

author: Josh

category: Articles

tags: Eucalyptus, Ubuntu

permalink:  diskless-eucalyptus-cluster
---
Overview
--------

**Platform(s):**Ubuntu 9.10 (Should work in 10.04+)

Ubuntu Enterprise Cloud (UEC) is a new initative by Ubuntu to create a
private cloud based on the Ubuntu architecture. A cloud is a huge
buzzword right now that pretty much means whatever you want it to. When
we are talking about Ubuntu Enterprise Cloud (UEC), it refers to a
cluster of computers that run virtual machines. Virtual machines are
basically just images of a normal computer/server made to run on top of
other servers, but using a cloud of virtual machines provides easier
management, failover, and better uptime if implemented properly. It also
can be "elastic", or allow sets of virtual machines to grown bigger and
smaller as they get more or less load on each individual server.

This tutorial covers how to install UEC, and make all the nodes, which
actually run the virtual machines, diskless. By making the nodes
diskless, all the files required to run them are centralized on a single
server, which makes managing them much easier. It also means that a new
server can be deployed by simply plugging them into the network, and
letting them network boot. Normally, you would have to install and
configure Ubuntu on each and every server, which becomes quite tedious
if you have more than a few servers.

So let's get down to business:

Installation
------------

To run UEC, you will need one computer to run as the Front End server
and one or more nodes. The Front End runs the cloud controller, cluster
controller, storage controller, and walrus (a storage service similar
to [Amazon's S3 service](http://aws.amazon.com/s3/).) You can check the
UEC page for a more in-depth explanation of each of these tecnologies.
According to the UEC wiki, these are the recommended requirements for
the Front End:

**Front End Requirements**

+----------------+---------------+-----------------+-------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
**Hardware** **Minimum** **Suggested** **My Setup** **Notes** |
+----------------+---------------+-----------------+-------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
CPU 1GHZ 2 x 2GHz 933MHz For an all-in-one front end, it helps to have
at least a dual core processor |
+----------------+---------------+-----------------+-------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
Memory 512MB 2GB 768MB The Java web front end benefits from lots of
available memory |
+----------------+---------------+-----------------+-------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
Disk 5400rpm IDE 7200rpm SATA 4x10000rpm SCSI in RAID10 Slow disks will
work, but will yield much longer instance startup times |
+----------------+---------------+-----------------+-------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
Disk Space 40GB 200GB 146GB (protected by RAID10) 40GB is only enough
space for a single image, cache, etc., Eucalyptus does *not* like to run
out of disk space |
+----------------+---------------+-----------------+-------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
Networking 100Mbps 1000Mbps 100Mbps Machine images are hundreds of MB,
and need to be copied over the network to nodes (We will try to solve
this by centralizing all the disk work) |
+----------------+---------------+-----------------+-------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+

The node runs the node controller software, for running virtual
machines.

**Node Requirements**

+----------------+-----------------+-------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
**Hardware** **Minimum** **Suggested** **My Setup** **Notes** |
+----------------+-----------------+-------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
CPU VT extensions VT, 64-bit, Multicore AMD Black 5000+, VT For an
all-in-one front end, it helps to have at least a dual core processor
(VT extensions speed up virtual machines, but not essential for testing
environment) |
+----------------+-----------------+-------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
Memory 1GB 4GB 4GB Additional memory means more, and larger guests |
+----------------+-----------------+-------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
Disk 5400rpm IDE 7200rpm SATA or SCSI 5400rpm IDE Eucalyptus nodes are
disk-intensive; I/O wait will likely be the performance bottleneck (This
is solved with the diskless setup. I have disks for /tmp and swap space
on nodes) |
+----------------+-----------------+-------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
Disk Space 40GB 100GB 80GB Images will be cached locally (on the server
for diskless.) Eucalyptus does *not* like to run out of disk space |
+----------------+-----------------+-------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
Networking 100Mbps 1000Mbps 100Mbps\~ Machine images are hundreds of MB,
and need to be copied over the network to nodes (or between disks in our
case) |
+----------------+-----------------+-------------------------+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

If your processor doesn't include VT extensions, it seems you can use
Xen instead of KVM (virtualization technologies that Eucalyptus relies
on, KVM is the UEC default). I believe you would have to use Xen for all
nodes then, and may have unreliable and slower performance.  Check this
link for more <info:%C2%A0>[Running Xen on Ubuntu Intrepid and
Jaunty](http://www.infohit.net/blog/post/running-xen-on-ubuntu-intrepid-and-jaunty.html).
The link refers to Ubuntu 8.10 and 9.04, instead of 9.10 as we are
using.  Leave me a comment with your experiences if you use this please!

**Installing From CD**

1.[Download the Ubuntu 9.10 Server
ISO](http://www.ubuntu.com/getubuntu/download-server), burn it to a CD,
and boot off it on the computer you intend to be the Front End 2. When
you boot, select "Install Ubuntu Enterprise Cloud" |private1-cr.png| 3.
Choose "Cluster" when asked whether to install "Cluster" or "Node".
|cluster-node-cr.png| 4. Answer the following questions when asked: a.
Name your cluster ("HomeOne" in my case) b. The range of IP's to assign
to virtual machines (192.168.1.201-192.168.1.239 in my case). Make sure
these numbers aren't being assigned by any other DHCP server (like your
router). I have my whole cluster of servers behind a second router, to
keep everything isolated. c. Choose local configuration for Postfix,
unless you need to set this up to actually send internet mail.

Once it finishes the install, login and begin configuration.

**Front End Configuration**

Update the system:

{% highlight bash %}
sudo apt-get -y update && apt-get -y dist-upgrade
{% endhighlight %}

Install necessary packages:

{% highlight bash %}
sudo apt-get -y install dnsmasq syslinux nfs-kernel-server initramfs-tools
{% endhighlight %}

(Other guides will tell you to use dhcp3-server and tftpd-hpa in place
of dnsmasq, but I find dnsmasq much simpler to use in a small network.
If you're going for a large deployment with lots of customization, I
would advise following the link at the bottom of the page the the Ubuntu
Community documentation so you can use the other suggested packages.)

Set a static IP

{% highlight bash %}
sudo nano /etc/network/interfaces
{% endhighlight %}

Change "iface eth0 inet dhcp" to:

{% highlight bash %}
iface eth0 inet static
address 192.168.1.10
netmask 255.255.255.0
network 192.168.1.0
broadcast 192.168.1.255
gateway 192.168.1.1
{% endhighlight %}

Remove dhcp3-client to let the update go into effect

{% highlight bash %}
sudo apt-get -y remove dhcp3-client
{% endhighlight %}

Restart networking

{% highlight bash %}
sudo /etc/init.d/networking restart
{% endhighlight %}

For some reason, I couldn't log in to the Eucalyptus web interface after
this, so I uninstalled and reinstalled, which fixed it. First though,
try

{% highlight bash %}
sudo service eucalyptus restart  
sudo service eucalyptus-cloud restart
{% endhighlight %}

If that fails, uninstall and reinstall

{% highlight bash %}
sudo apt-get -y remove eucalyptus-ccsudo apt-get -y install eucalyptus-cc
{% endhighlight %}

Edit dnsmasq.conf

{% highlight bash %}
sudo nano /etc/dnsmasq.conf


#Makes dnsmasq the authoritative dhcp server for the network (instead of any router), then tells what IP's it

#is allowed to hand out. Don't use with another router on the network (could be very bad!)
dhcp-authoritative
dhcp-range=192.168.1.21,192.168.1.99,6h

#Enables the builtin TFTP server, tells it where to look, and what file to use
enable-tftp
tftp-root=/opt/uec
dhcp-boot=pxelinux.0

#Assigns certain MAC address a specific IP and hostname

dhcp-host=00:00:00:00:00:00,cloud-dl380,192.168.1.11
dhcp-host=00:00:00:00:00:00,cloud-rioworks,192.168.1.12
{% endhighlight %}

You can find the MAC address if you boot up each server, and tell it to
network boot. It will display the MAC/Hardware address. You can find the
MAC address if you boot up each server, and tell it to network boot.  It
will display the MAC/Hardware address. Replace the 00:00:00:00:00:00:00
with the actual numbers of each server. This isn't absolutely necessary,
but I find it is extremely helpful in keeping track of what is going on.

{% highlight bash %}
sudo /etc/init.d/dnsmasq restart
{% endhighlight %}

Configure TFTP

{% highlight bash %}
sudo mkdir -p /opt/uec/pxelinux.cfg
sudo cp /usr/lib/syslinux/pxelinux.0 /opt/uec
sudo nano /opt/uec/pxelinux.cfg/default


LABEL linux
    KERNEL vmlinuz-2.6.31-14-generic-pae
    APPEND root=/dev/nfs initrd=initrd.img-2.6.31-14-generic-pae nfsroot=192.168.2.2:/opt/nfsroot ip=dhcp rw
{% endhighlight %}

You'll have to replace vmlinux and initrd.img with the appropriate
versions based on your systems' kernel. These will be found later.

Make the directory world writable, to avoid errors later. This could be
a security concern.

{% highlight bash %}
sudo chmod 777 /opt/uec
{% endhighlight %}

Prepare NFS directory

{% highlight bash %}
sudo mkdir /opt/nfsroot
{% endhighlight %}

Edit /etc/exports to include the new directory

{% highlight bash %}
sudo nano /etc/exports

/opt/nfsroot *(rw,no_root_squash,async)
{% endhighlight %}

Note: This exports the entire NFS volume to anyone on your network with
an address 192.168.1.\*\*\* that wants to mount it. NFSv4 provides more
security features, which could be useful but were not implemented here.
I manually specified each IP address for a bit better security, but a
diligent attacker could easily get around this. To do this, just make
one line per client, and specify the final numbers instead of \*\*\*
like this:

{% highlight bash %}
/opt/nfsroot 192.168.1.11(rw,no_root_squash,async)
/opt/nfsroot 192.168.1.12(rw,no_root_squash,async)
{% endhighlight %}

Now export the NFS directory

{% highlight bash %}
exportfs -rv
{% endhighlight %}

Next we need to fill the NFS directory with the files the node needs to
run.

**Node Configuration**

The easiest way I found was to install Ubuntu on one of the systems you
want to use as a diskless node, copy the files over to the NFS
directory, and then restart.

I believe there is a way to turn the NFS directory into a chroot, and
use Ubuntu's built in tools to install it directly to the directory, and
use no node system to install on. The tool to do this is debbootconf,
but most tutorials I found are for much older Ubuntu versions. I'm
working on a way to do this right now, and when it is finished and
tested, I will put together a script to run on the server, to do this
whole tutorial with only two commands.

**Install on Node Method**

Install from CD using the same instructions as on the cloud controller,
except select node instead of cluster.

Update the system

{% highlight bash %}
sudo apt-get -y update && apt-get -y upgrade
{% endhighlight %}

Install NFS client package

{% highlight bash %}
sudo apt-get -y install nfs-common
{% endhighlight %}

Set a temporary password for the eucalyptus user, so we can link the
node to the Front End

{% highlight bash %}
sudo passwd eucalyptus
{% endhighlight %}

**Prepare the initrd and kernel for PXE booting**

Copy the kernel to your home (\~) directory, so it is easy to get at
later.

{% highlight bash %}
cp /boot/vmlinuz-`uname -r` ~
{% endhighlight %}

(The above uses a cool trick of embedding a command in a command. Uname
-r prints the kernel version number. This is useful if you have multiple
kernel versions available for nodes to boot)

Now create the initrd image

{% highlight bash %}
sudo nano /etc/initramfs-tools/initramfs.conf
{% endhighlight %}

Change the lines to this:

{% highlight bash %}
MODULE=netboot
BOOT=nfs
{% endhighlight %}

Replace the numbers with whatever your kernel version is

{% highlight bash %}
sudo mkinitramfs -o /home/<USERNAME>/initrd.img-2.6.31-14-generic-pae
{% endhighlight %}

Now copy over the files you've installed to the NFS share.

{% highlight bash %}
mount -t nfs -o nolock 192.168.1.10:/opt/nfsroot /mnt
cp -ax /. /mnt/.
cp -ax /dev/. /mnt/dev/.
{% endhighlight %}

**Modify NFS image for netbooting** First we need to change network
interfaces to reflect the netbooting. You don't want Ubuntu to try to
reconfigure the network, after it has already been configured while
loading the PXE image, so we need to disable DHCP on the first
interface.

{% highlight bash %}
nano /opt/nfsroot/etc/network/interfaces
{% endhighlight %}

Delete "auto eth0" and change "iface eth0 inet dhcp" to "iface eth0 inet
manual" Modify "bridge_ports eth0" to "bridge_ports eth1"

Because all the files are shared between all the nodes, certain things
will be the same on all the servers. One thing we want to be different
on all the nodes is their hostname, which is controlled by the
/etc/hostname file. We previously set up hostnames to be delegated by
the Cloud Controller via dnsmasq based on each MAC address. However, if
the hostname file exists, Ubuntu ignores these assignments. The easiest
way to fix this is to change /etc/hostname to /etc/hostname.bak and
delete the original

{% highlight bash %}
sudo cp /etc/hostname /etc/hostname.bak
sudo rm /etc/hostname
{% endhighlight %}

Now each node will use the hostname we specify.

We need to edit where the filesystems that we copied to the server are
mounted and how. We are going to mount a few of the directories as
tmpfs, so they can be on each of the hosts, and prevent certain files
from being opened by multiple nodes at the same time (which could
corrupt them, or just not work). We are also mounting proc, which is the
processor filesystem (kind of important!) and mounting the NFS share as
the root of our drive.

{% highlight bash %}
nano /opt/nfsroot/etc/fstab

# /etc/fstab: static file system information.#
# <file system> <mount point> <type> <options> <dump> <pass>
proc /proc proc defaults 0 0
/dev/nfs / nfs defaults 1 1
none /tmp tmpfs defaults 0 0
none /var/run tmpfs defaults 0 0
none /var/lock tmpfs defaults 0 0
none /var/tmp tmpfs defaults 0 0 /dev/hdc /media/cdrom0 udf,iso9660 user,noauto 0 0
{% endhighlight %}

This last line is only if you want your CD drive to be accessible. I've
never needed it on these nodes (most don't even have a CD drive to save
on power requirements) but it might be something you want.

Now reboot your test node, remove the CD, and hope it works. (It
should!)

{% highlight bash %}
sudo shutdown -r now
{% endhighlight %}

(Don't forget to change to network boot in BIOS. This can usually be
found as Network Boot or PXE Boot, maybe even LAN Boot. You'll need to
put it to put it above your harddrive in the Boot Priority section. I
usually just put it at the top of the list.)

If all went well, your system will reboot and look exactly like it did
before, except with a new hostname and no required harddrive.

Now we just have to go about connecting the node to the server. This is
a simple process. On the server, after all the nodes you want running
are started (or the first one just to test), run:

{% highlight bash %}
sudo euca_conf --no-rsync --discover-nodes
{% endhighlight %}

It should register all the nodes automatically.

If it doesn't check out the UEC Node Installation page UEC Node
Installation page, and verify all your settings. It worked for me every
time, so you shouldn't have an issue.

You should SSH into one of the nodes and change the password of the
eucalyptus user back to no password.

{% highlight bash %}
ssh eucalyptus@NODEHOSTNAME
sudo passwd -d eucalyptus
exit
{% endhighlight %}

**Configuring the Cluster**

In your web browser, go tohttps://192.168.1.10:8443_

The default username/password is admin/admin. Once you login, you will
be prompted to put in a new password (please use something strong, with
at least one uppercase, lowercase, number, and symbol, at least 8
characters long.)

|image2|

Go to the Configuration tab, and change all hosts to 192.168.1.10. Click
"Save Configuration" after each configuration.

If you want an even easier management interface, you can go to the
Services tab, and register with RightScale. You will need a RightScale
account (free for the basic account), and you need to change
192.168.1.10 to either your public IP or a dynamic DNS hostname from
companies like DynDNS.org. I find this to be a very useful service, but
not everyone really needs it. They provide a good way to automate
installation and configuration of your virtual machines. You need to
forward ports on your router for this (8443 and 8774) to your Front End
server.

In the Extras tab, there is a link at the bottom to tools you can use to
configure and control your cluster if you don't use RightScale.

You can download pre-configured images in the Images tab. These are
great starting points. For more information on images, check
the [Eucalyptus Administrator's
Guide](http://open.eucalyptus.com/wiki/EucalyptusImageManagement_v1.6).
They have done a fantastic job with giving great step by step
instructions.

Sometime soon I'll be rolling this all up into a script, because that is
how I am. If anything goes wrong, I don't want to have to follow my own
tutorial when I'm setting one of these two servers up again. Also, if
anyone has a good tutorial on how to do this with debbootstrap, I would
be very appreciative. Then this could all be one script run on the
server.

**Thanks** Thanks to:

The Contributors to the Ubuntu Documentation wiki, especially the ones
at:

<https://help.ubuntu.com/community/DisklessUbuntuHowto>

<https://help.ubuntu.com/community/UEC/CDInstall>

And thanks to all the dedicated people who have made Ubuntu and UEC as
amazing as it is today.

This work is covered under the [Creative Commons Attribution-Share Alike
3.0 license.](http://creativecommons.org/licenses/by-sa/3.0/) Basically,
you can use this text in anyway you want, as long as it is covered under
the same or similar license, and you mention this site (and the Ubuntu
Wiki), as sources. Enjoy!
