---
title: Minecraft Server On Ubuntu 12.04


date: 2013-06-04 15:02

author: Josh

layout: default
category: Articles

tags: Minecraft, Ubuntu

permalink: minecraft-server-on-ubuntu-12-04
---

So I finally gave in and joined the craze that is Minecraft. It is a
highly addictive game, so I would highly recommend against playing it
while you have stuff to do. A couple people that I work with love the
game, and I love servers, so I figured we should set up our own server!
I am hosting the server on an [OVH dedicated
server](http://www.ovh.com/us/dedicated-servers/). Here's how you can
set up your own server, along with some tips I've found to make life
much easier.

Considerations
--------------

Before you set up a server, you need to consider a few things, and
decide whether it is worth it to host it in your house or just rent one
from one of the many places selling Minecraft hosting.

1.  Power - This server probably needs to be on 24/7, and will use a lot
    of power. You might be able to get away with a smaller, Intel Atom
    based server, such as one by SuperMicro, which will use less power,
    but won't be able to host as many people.
2.  Heat - I had mine in a closet, which is not ideal. The closet is
    right next to an air conditioner, so that helps with heat, but the
    closet is still really hot and is probably shortening the life span
    of this and the other servers.
3.  Internet Connection - Research online shows each player probably
    uses about 20Kb/s. So take about 80% of your upload (not download)
    speeds, and divide by number of players. You may want to lower the
    number of players a bit, since each player loads a chunk of the map
    when they connect, which will use more bandwidth ideally, to allow
    users to log in quickly.
4.  RAM - each user will probably use about 64-128MB of RAM. The system
    itself will use anywhere around 100MB, but should have room to grow
    a bit.

I ran the server out of my house for about a year, but found it easier
to migrate it to a dedicated server. At \$40/mo (and I use the server
for a bunch of other things), it is a pretty great deal.

If you're OK with all this, move on!

Installation
------------

If you're using your own server, go grab a Ubuntu server image (I'm
using 12.04 x64, since it will be supported until 2017), and burn it to
CD. Boot off it, and do a default configuration, making sure to check
OpenSSH server at the end.

Minecraft is a Java app. Ubuntu comes with the OpenJDK, which doesn't
work well with Minecraft. Let's download the official version, and set
that as the default for Java. We're going to use Java 7, because Java 6
is End of Life.

{% highlight bash %}
# Run updates
sudo apt-get -y -q update &amp;&amp; sudo apt-get -y -q upgrade
# Install tmux so we can keep our server running indefinitely and easily send it commands
sudo apt-get install tmux
# Get some software so we can add the repository with Java in it, then update again
sudo apt-get install python-software-properties
# Add the repository with Java in it, then update to get a list of packages
sudo add-apt-repository ppa:eugenesan/java
sudo apt-get update

# Finally, install Java! Agree on the pink screen that pops up.
sudo apt-get install oracle-java7-installer
{% endhighlight %}

We're going to create a separate user to run our Minecraft server
through. We will give this user limited permissions, so if anyone breaks
into the server somehow through Minecraft, they won't have root access
to the rest of the server. When managing Minecraft, you'll log in
through this user.

{% highlight bash %}
# Enter a password and the rest of the info when prompted.
sudo adduser minecraft
# Change to the minecraft user
su minecraft
# Go to the minecraft home directory (/home/minecraft)
cd ~
{% endhighlight %}

Now that we have Java installed, we can get the client. You can either
go to Minecraft.net and download it, or you can just download it from
the command line. We've already got the command line open, so let's use
that. We're going to put it in the games folder so other users can
easily access Minecraft also.

{% highlight bash %}
mkdir minecraft
cd minecraft
wget https://s3.amazonaws.com/MinecraftDownload/launcher/minecraft_server.jar
{% endhighlight %}

Now we are going to add some commands that will make our lives as
Mincraft server admins easier.

{% highlight bash %}
# As minecraft user
nano ~/.bash_aliases
----
# Add these lines
alias startmc='tmux new -d -s &quot;minecraft&quot; &quot;java -Xincgc -Xms1024M -Xmx1024M -jar /home/minecraft/minecraft/minecraft_server.jar nogui&quot;'
alias mcconsole='tmux attach-session -d -t minecraft'
alias mclog='less /home/minecraft/minecraft/server.log'&quot;
# Type ctrl + X then y to save and exit nano

# To load these new settings, we need to reload bash.
bash

# And the aliases should be working now!
{% endhighlight %}

These commands use a program called tmux. Basically, if you were to just
run the java command they have on the wiki, the server would run only as
long as you were connected to the server (and later I'll show you how to
connect via SSH, where you wouldn't want that connected all the time).
This way, it is run in its own process and not tied to the terminal you
have open. The mcconsole command simply attaches the screen to your
current terminal.

Now, these three commands should be available at the terminal. So type
"startmc" to start a Minecraft server inside of a tmux session (so you
can connect to it later). "mcconsole" will bring up the Minecraft
console for you to command. Hit Ctrl+b then d will detach the tmux
session, and put you back at the normal command line. "mclog" will
display the server logs, so you can read through.

Make It Public (For Home Servers)
---------------------------------

First, we need to know our home server's IP address. As your normal user
(not minecraft user), run:

{% highlight bash %}
# We're looking for an address to the effect of 192.168.###.###.
ifconfig \grep 'inet addr:'
{% endhighlight %}

What good is a server if only you can access it? Boo! This is going to
be general, because I can't cover every possible router in the world.
Grab that number that you kept track of, which is your machine's IP
address. Log into your router, and look for NAT or Port Fowarding. Set
up a port forward on port 22 to port 22 on the IP address you wrote
down. This will allow us to log in and manage the server from anywhere.
Also, we need to forward port 25565 to that IP address also.

If your router supports a dynamic DNS name, I would highly recommend
setting that up, probably through DynDns.org. This way your users can
connect to a domain name like minecraft.example.com, rather than an IP
address, which may change, depending on your ISP. To get your public IP
address, simply visit <http://www.whatismyip.com/>.

Connecting To It
----------------

If you want to connect for gaming, simply fire up Minecraft and type in
either your public IP address, or the dynamic domain name you set up. If
you want to connect via SSH to manage the server, either use SSH
(Linux/Mac), or [download
Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).
Simply connect to your public IP or your dynamic DNS name, and run
"mcconsole" to get to the console.

Conclusion
----------

I hope this worked for you. If it didn't, post a note in the comments
and I will try to help you. If it did, post that too!! I like knowing
that I'm helping. If you have suggestions, please post those so everyone
can run this awesome game better.
