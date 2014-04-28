---
title: FreeNAS to Ubuntu - Torrent Server (Transmission)


date: 2012-02-12 13:27

author: Josh

layout: default
category: Articles

tags: FreeNAS, FreeNAS to Ubuntu, Torrent, Ubuntu

permalink: freenas-to-ubuntu-torrent-server-transmission
---

If you're coming from FreeNAS 7 to Ubuntu, you remember the very handy
torrent server that was built in. Well we can exactly replicate that
with Transmission's and its WebGUI. A torrent server provides you a
great way to download torrents directly to your server. It also allows
you to constantly seed torrents, even when your main computers are off.
I use this to host copies of OS's (such as Ubuntu!) so other people can
download them faster.

**Note:**This is an article in the series [FreeNAS to
Ubuntu](http://www.servercobra.com/tag/freenas-to-ubuntu/). Check out
the article for the [initial Ubuntu fileserver setup coming from
FreeNAS](http://www.servercobra.com/replacing-freenas-with-ubuntu-file-server/).

First we need to install Transmission.

{% highlight bash %}
sudo apt-get install -y transmission-daemon
{% endhighlight %}

Transmission's settings are saved in a JSON file. We'll edit that to
update the settings appropriately. We're going to enable the remote
interface, allow all computers connect to it, change the password (which
will be encrypted when we reload the settings), and change the
download-dir and incomplete-dir. First, let's stop transmission. For
some reason, it occasionally overwrites the settings when restarting.

{% highlight bash %}
sudo service transmission-daemon stop
sudo nano /var/lib/transmission-daemon/info/settings.json
----
# Change:
# "rpc-enabled": false,
"rpc-enabled": true,

# "rpc-password": "$kljfkljwerjwauiouak438908",
"rpc-password": "plaintext-new-password",

# "rpc-whitelist-enabled": true,
"rpc-whitelist-enabled": false,

# "download-dir": "",
"download-dir": "/mnt/storage/Torrents/",
# "incomplete-dir-enabled": false,
"incomplete-dir-enabled": true,
# "incomplete-dir": "",
"incomplete-dir": "/mnt/storage/Torrents/incomplete",
{% endhighlight %}

Now we need to create the appropriate directories, and give them proper
permissions. We are going to own the directory to the Transmission user
and group (debian-transmission), give that user:group full permissions,
and then give read permissions to the world (the rest of the users). You
could alternately add yourself to the to the debian-transmission group,
so that you have write and execute permissions, but for this tutorial,
we're going to share it out via Samba, so read is all we'll need.

{% highlight bash %}
mkdir -p /mnt/storage/Torrents/incomplete/

chown -R debian-transmission:debian-transmission /mnt/storage/Torrents/
chmod -R 774 /mnt/storage/Torrents/
{% endhighlight %}

Now that everything is ready, we need to restart the server to encrypt
the password and reload the settings.

{% highlight bash %}
sudo service transmission-daemon start
{% endhighlight %}

Now you should be able to connect to your server by visiting
"hostname:9091". You can set up port forwarding on your router so you
can connect when you're away from home, by forwarding a port to your
server's IP and port 9091. This port can be changed in the transmission
config file. The setting is called ""rpc-port": 9091,".
