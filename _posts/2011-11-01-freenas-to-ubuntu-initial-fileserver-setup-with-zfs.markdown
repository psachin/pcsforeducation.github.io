---
title: FreeNAS to Ubuntu - Initial Fileserver Setup with ZFS


date: 2011-11-01 13:03

author: Josh

category: Articles

tags: FreeNAS, FreeNAS to Ubuntu, Ubuntu, ZFS

permalink: freenas-to-ubuntu-initial-fileserver-setup-with-zfs
---

**Background: **I have always loved FreeNAS. It was one of the first
servers I ever set up, back in the day in my parent's basement. It was
running on a Pentium 3, 933MHz, 256MB of RAM, and about 500GB of storage
space. Eventually, it got upgraded to a real piece of server hardware,
more drives, etc. When they announced the buyout by iX Systems and the
upgrade to version 8 with a complete rewrite, I was skeptical. I've
tried their new software, and I wasn't overly impressed. I've been
having various issues on v7 as well, mostly related to ZFS and random
crashes. v8 is much more enterprise oriented, and dropped the fun bits
for home servers, such as torrent server and UPnP, for media streaming.
I bought 4 new 2TB drives and an external enclosure, and tried to
reinstall FreeNAS to start fresh. No such luck, and after a few hours of
crashes on start, I gave up and decided, it's time to go with Ubuntu,
like the rest of my house, with ZFS of course.

**Goals:**To replace FreeNAS, there are a few pieces of software that
absolutely have to work. Mainly, they'd be Samba for Windows sharing,
NFS for ESXi and Linux sharing, ZFS for the storage pool (file system
and redundancy), a web server to stream/download files in the pools, and
preferably some type of web administration software. Nice things would
be UPnP that actually works with my Xbox (FreeNAS never did), and on the
fly encoding (maybe DivX?) to stream my HD movies over the Internet when
I'm not at home. We'll get a basic version up in this tutorial, and then
the more advanced stuff in the next one.

**Installation:**Install Ubuntu server like normal. Since this is my
main file server, I'm going with Ubuntu 10.04. I don't feel like
upgrading this every 18 months when support runs out. Since 10.04 is a
LTS (Long Term Service) release, it will be supported until April 2015.
You'll also want the 64 bit edition, because ZFS on Linux is better
supported by 64 bit. Install like normal. I'm using one drive as the OS
disk, and the rest will go into zpools. The only thing to make sure you
do is to install OpenSSH server when it asks which server software to
install.

**Configuration:** First off, let's update the server.

{% highlight bash %}
sudo apt-get -y -q update && sudo apt-get -y -q upgrade
{% endhighlight %}

Next, let's get ZFS installed, since most everything else relies on it.
There is a project called ZFS on Linux, which provides a port of the ZFS
kernel module for Linux. It can't be included directly in the Linux
kernel because of incompatible licenses, but we can put it in ourselves.
Versions below 0.60 won't let you mount the ZFS pools (serious problem
right?). [STRIKEOUT:So we will install the 0.60rc6 straight from the git
repo where it is being actively developed. Hold onto your seat! We're
going to be compiling directly from source.] As rj in the comments
pointed out, there is now a PPA with ZFS on Ubuntu, so we'll use that
method. You can still get the newest code straight from the repo and
compile if you are having problems.

{% highlight bash %}
sudo apt-get -y install build-essential gawk zlib1g-dev uuid-dev vim-nox python-software-properties
sudo add-apt-repository ppa:zfs-native/stable
sudo apt-get update
sudo apt-get install ubuntu-zfs

# Add zfs modules to be loaded at boot
sudo nano /etc/modules
---
# Add on:
spl
zavl
znvpair
zunicode
zcommon
zfs
---

# Incorporate new modules into the boot files
sudo update-initramfs -u

# If you aren't using the PPA (installing from source), you'll need your own init script, so ZFS pools are mounted at startup. The script (from the PPA) looks like this:
sudo nano /etc/init.d/zfs
-----
#!/bin/sh
#
# chkconfig: 2345 01 99
#
### BEGIN INIT INFO
# Provides: zvol zfs
# Required-Start: $local_fs
# Required-Stop: $local_fs
# Required-Start:
# Required-Stop:
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: Start/stop ZFS subsystem.
# Description: ZFS is an advanced filesystem designed to simplify managing
#              and protecting your data.  This service mounts the ZFS
#              filesystems and starts all related zfs services.
### END INIT INFO

PATH=/sbin:/bin:/usr/sbin:/usr/bin

. /lib/lsb/init-functions
. /lib/init/vars.sh

[ -f /etc/default/zfs ] && . /etc/default/zfs

do_start() {
        log_begin_msg "Starting ZFS subsystem"
        log_progress_msg "filesystems"
        zfs mount -a
        RET=$?
        if [ $RET != 0 ] ; then
                log_end_msg $RET
                exit $RET
        fi
        log_end_msg 0

        log_begin_msg "Exporting ZFS filesystems"
        zfs share -a
        log_end_msg $?
}

do_stop() {
        log_begin_msg "Stopping ZFS subsystem"
        log_progress_msg "filesystems"
        zfs umount -a
        RET=$?
        if [ $RET != 0 ] ; then
                log_end_msg $RET
        fi
        log_end_msg 0
}

do_status() {
        zpool status
        echo ''
        zpool list
        true
}

case "$1" in
  start)
        [ -z "$ZFS_MOUNT" ] && exit 0
        do_start
        ;;
  stop)
        [ -z "$ZFS_UNMOUNT" ] && exit 0
        do_stop
        ;;
  status)
        do_status
        ;;
  *)
        [ -n "$1" ] && echo "Error: Unknown command $1."
        echo "Usage: $0 {start|stop|status}"
        exit 3
        ;;
esac

-----
chmod 755 /etc/init.d/zfs
{% endhighlight %}

Now that we have ZFS installed, we either need to mount existing pools,
or create new pools. ZFS allows you to create pools with redundancy such
as mirroring, or RAIDZ(x), which allows x amount of drives to die
without losing data. For example RAIDZ can lose on drive, RAIDZ2 can
lose 2, and RAIDZ3 can lose 3.

Note: all ZFS and ZPool commands must be run as root/sudo

{% highlight bash %}
# Let's create a RAIDZ (like RAID 5) pool named storage from 3 disks
sudo zpool create storage raidz /dev/sdb /dev/sdc /dev/sdd

# Import an existing pool named storage (perhaps created on a previous FreeNAS box)
sudo zpool import storage

# And then we can create filesystems on the pool, much like folders. We already have a root folder at /mnt/storage/
sudo zfs create storage/Movies
sudo zfs create storage/Music
sudo zfs create storage/Upload

# Now your ZFS should look like this:
# /mnt/storage
# /Movies
# /Music
# /Upload
{% endhighlight %}

Now that we have the server's pools in order, we need to provide a way
to upload/download data. We will start by using SMB, which allows
Windows, Linux, and Macs connect. It also allows us to provide public
read-only shares, public writable shares, and password protected shares.

{% highlight bash %}
# Prepare sharing
sudo apt-get -y install samba nfs-kernel-server
sudo nano /etc/samba/smb.conf
---
# Change
# security = user
# to
security = user

# No password, read only share
[sharename]
comment = This is a share
path = /mnt/storage/share
browsable = yes
guest ok = yes
read only = yes
create mask = 0755

# No password, writable share
[writableshare]
comment = This is a writable share
path = /mnt/storage/writable
guest ok = yes
writeable = True
create mask = 0744

# Password protected share
[passwordsharename]
comment = This share requires a password
path = /mnt/storage/secret
browsable = yes
guest ok = no
read only = yes
create mask = 0755
user = username otheruser

# To allow users to connect to password protected shares, run smbpasswd to generate the login password
sudo  smbpasswd -a username
sudo smbpasswd  -a otheruser
{% endhighlight %}

In my case, this worked without restarting Samba, however it might be a
good idea anyway.

{% highlight bash %}
sudo reload smbd
{% endhighlight %}

You should now be able to connect to the server by going to
"<smb://serverip>" in Nautilus or "\\serverip" in Windows Explorer.
Add as many shares as you need.

This gives you a very basic file server. If you want to get a torrent
server similar to the FreeNAS one, you can check out the [Ubuntu Torrent
Server](http://www.servercobra.com/freenas-to-ubuntu-torrent-server-transmission/)
article.

In my next articles, I will explore using NFS to share data to Linux
clients, a torrent server, UPnP for streaming to XBox 360/PS3, and a web
server to share files over the Internet (hopefully with on the fly
encoding), and maybe those sweet beeps that FreeNAS makes on
startup/shutdown.

If anything in the tutorial doesn't work for you, make sure to leave a
comment and I'll try to fix it! "It worked!" comments are also
appreciated!
