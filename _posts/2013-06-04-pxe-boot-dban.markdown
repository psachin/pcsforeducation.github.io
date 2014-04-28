---
title: PXE Boot DBAN


date: 2013-06-04 20:40

author: Josh

layout: default
category: Articles

tags: DBAN, PXE, Ubuntu

permalink: pxe-boot-dban
---

At the University I work for, we make sure to DBAN all of our machines
before they are sold off for security reasons. We used to pull all the
machines, put them in a pile, and go to each machine, plugging in a
power cable, video cable, boot off a CD, then moving the CD and video
cable to the next one. It took forever and was a pain. So I decided to
make it easy to boot DBAN via PXE. All of our Windows machines reboot
each night, and have PXE set as their first boot device. The night
before we replace machines, I switch them to boot from the DBAN server,
and by the time I get there in the morning, they are finished wiping.

This weekend, a buddy of mine needed to DBAN a huge pile of computers
before donating them to needy people. After replicating the server I
built at the university, we decided I should probably throw up a blog
post about how to do it.

For this machine, I'm going to set up a local switch on a secondary
Ethernet port so I can DBAN machines in my office. We'll need a DHCP
server for the local network, and a TFTP server for both the local
switch and the rest of the network.

To start, I installed Ubuntu 12.04 x64 on a machine. I did the default
install, checking OpenSSH server at the end.

First, let's get the machine up to date, and install the software we
need. We're going to use dnsmasq as our DHCP server (it has a very
simple configuration) and tftpd-hpa for our TFTP server. dnsmasq has a
builtin TFTP server, but as far as I could tell, you can't disable DHCP
and not disable TFTP for an interface. While I'm pretty we have enough
safeguards to prevent a rogue DHCP server from assigning addresses (and
PXE files), I really don't want to be responsible for half our network
getting DBANed.

{% highlight bash %}
sudo apt-get -y update && sudo apt-get -y upgrade
sudo apt-get -y install dnsmasq tftpd-hpa
{% endhighlight %}

First, we're going to set up dnsmasq. We're going to make a pool of
addresses for the local network switch on eth1. We're going to set eth1
to a static address. I set my server name to dban-server, but you can
adjust yours accordingly.

{% highlight bash %}
sudo nano /etc/network/interfaces
--
# Add this to the bottom of your interfaces file.
auto eth1
iface eth1 inet static
address 10.0.0.1
netmask 255.255.255.0
--

sudo nano /etc/dnsmasq.conf
----
# Add these to the top of the file
dhcp-range=10.0.0.2,10.0.0.254,6h
dhcp-boot=pxelinux.0,dban-server,10.0.0.1
interface=eth1
--

# Restart the affected services
sudo /etc/init.d/networking restart
sudo /etc/init.d/dnsmasq restart
{% endhighlight %}

Now, we need to set up tftpd-hpa for both interfaces. I was having
problems with the tftp server starting up on boot. The included upstart
script seemed to start up before networking was ready, and then it would
fail. So we'll fix that up, and the default config file
(/etc/default/tftpd-hpa) will work just fine.

{% highlight bash %}
sudo nano /etc/init/tftpd-hpa.conf
--
# tftp-hpa - trivial ftp server

description "tftp-hpa server"
author "Chuck Short <zulcss@ubuntu.com>"

# Here, we change the upstart script to wait for the interface to be up
start on (local-filesystems and net-device-up IFACE=eth0)
# start on runlevel [2345]
# End changes.
--
# Restart the affected daemon
restart tftpd-hpa
{% endhighlight %}

Finally, we need to install the bootable files into /var/lib/tftpboot,
the default place tftpd-hpa looks for PXE files. We'll download DBAN,
extract the files, and put the standard PXE files into the directory to
make booting work.

{% highlight bash %}
# Download DBAN iso file
wget -O /tmp/dban.iso http://sourceforge.net/projects/dban/files/dban/dban-2.2.7/dban-2.2.7_i586.iso/download
sudo mount -o loop /tmp/dban.iso /mnt
sudo cp /mnt/\* /var/lib/tftpboot
cd /var/lib/tftpboot
# Download the pxelinux.0 file and a default configuration file with a single, default option to boot DBAN and start 'autonuke', which will automatically wipe all attached drives.
sudo wget http://mirrors.tummy.com/pub/ftp.ubuntulinux.org/ubuntu/dists/precise/main/installer-i386/current/images/netboot/ubuntu-installer/i386/pxelinux.0
sudo mkdir /var/lib/tftpboot/pxelinux.cfg
sudo nano /var/lib/tftpboot/pxelinux.cfg/default
--
DEFAULT autonuke

LABEL autonuke
KERNEL dban.bzi
APPEND nuke="dwipe --autonuke" silent
--

# Lastly, make sure our clients can read the files.
sudo chmod -R 755 /var/lib/tftpboot/
{% endhighlight %}

There we go. Plug a machine or a switch into a eth1, boot it up, set it
to boot from the network, and the drives will be erased shortly!
