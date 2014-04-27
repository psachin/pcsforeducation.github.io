---
title: Minecraft Server on Ubuntu 10.04


date: 2011-06-08 12:09

author: Josh

category: Articles

tags: Minecraft, Ubuntu

permalink: minecraft-server-on-ubuntu
---

So I finally gave in and joined the craze that is Minecraft. It is a
highly addictive game, so I would highly recommend against playing it
while you have stuff to do, though you get 25% off while it is in beta,
so [go buy it](http://www.minecraft.net/prepurchase.jsp) (not an
affiliate link, just love the game). A couple people that I work with
love the game, and I love servers, so I figured we should set up our own
server! I grabbed some hardware (A Dell GX620 with a Pentium D and 4GB
of RAM), and got started. Here's how you can set up your own server,
along with some tips I've found to make life much easier.

Considerations
--------------

Before you set up a server, you need to consider a few things, and
decide whether it is worth it to host it in your house or just rent one
from one of the many places selling Minecraft hosting.

1.  Power - This server probably needs to be on 24/7, and will use a lot
    of power. You might be able to get away with a smaller, Intel Atom
    based server, such as one by SuperMicro, which will use less power,
    but won't be able to host as many people. I'd assume at least 10
    people could be simultaneously hosted on a single core Atom without
    too much trouble. Also, power costs money, so figure out how much
    money you'll be spending.
2.  Heat - I have mine in a closet, which is not ideal. The closet is
    right next to an air conditioner, so that works, but the closet is
    still really hot and is probably shortening the life span of this
    and the other servers.
3.  Internet Connection - Research online shows each player probably
    uses about 20Kb/s. So take about 80% of your upload (not download)
    speeds, and divide by number of players. You may want to lower the
    number of players a bit, since each player loads the whole map when
    they connect, which will use more bandwidth ideally, to allow users
    to log in quickly.
4.  RAM - each user will probably use about 64MB of RAM. The system
    itself will use anywhere around 100MB, but should have room to grow
    a bit.

If you're OK with all this, move on!

Installation
------------

Go grab a Ubuntu server image (I'm using 10.04 x64, since it will be
supported until 2013), and burn it to CD. Boot off it, and do a default
configuration, making sure to check OpenSSH server at the end.

First off, we are going to create a Minecraft user. This will allow us
to always connect as that user, and have our own user account. This
command will create a new user with a home directory. Then we'll add a
password so we can ssh in as them.

{% highlight bash %}
sudo useradd -s /bin/bash -m -U minecraft
sudo passwd minecraft
sudo adduser minecraft admin
{% endhighlight %}

Now we need to get the prerequisite software for Minecraft, namely,
Java.

{% highlight bash %}
# Run updates
sudo apt-get -y -q update && sudo apt-get -y -q upgrade
# Get some software so we can add the repository with Java in it, then update again
sudo apt-get install python-software-properties
sudo apt-get -y -q update
# Add the repository with Java in it, then update to get a list of packages
sudo add-apt-repository "deb http://archive.canonical.com/ lucid partner"
sudo apt-get -y -q update
# Finally, install Java!
sudo apt-get -y install sun-java6-jre sun-java6-plugin
sudo update-alternatives --config java
# Finally, we will need the screen software later
sudo apt-get -q -y install screen
{% endhighlight %}

Now we will log in as the minecraft user and install the server software

{% highlight bash %}
su minecraft
cd /home/minecraft
mkdir minecraft-server
cd minecraft-server
# Go to http://www.minecraft.net/download.jsp, right click "minecraft_server.jar", and copy link address, and replace the URL to get the latest version
wget http://www.minecraft.net/download/minecraft_server.jar?v=1307640603925
# We will need this later. Keep track of the number starting with 192.
ifconfig \| grep 'inet addr:'
{% endhighlight %}

Now we are going to add some commands that will make our lives as
Mincraft server admins easier.

{% highlight bash %}
# As minecraft user
nano ~/.bash_aliases
----
# Add these lines
alias startmc="screen -dmSL minecraft java -Xms1024M -Xmx1024M -jar /home/minecraft/minecraft-server/minecraft_server.jar nogui"
alias mcconsole="screen -d -r minecraft"
alias mclog="less /home/minecraft/minecraft-server/server.log"
# Type ctrl + X then y to save and exit nano

# To load these new settings, we need to reload bash.
bash

# And the aliases should be working now!
{% endhighlight %}

These commands use a program called screen. Basically, if you were to
just run the java command they have on the wiki, the server would run
only as long as you were connected to the server (and later I'll show
you how to connect via SSH, where you wouldn't want that connected all
the time). This way, it is run in its own process and not tied to the
terminal you have open. The console command simply attaches the screen
to your current terminal.

Now, these three commands should be available at the terminal. So type
"startmc" to start a Minecraft server inside of a screen (so you can
connect to it later). "mcconsole" will bring up the Minecraft console
for you to command. Hit Ctrl+a then d will detach the screen, and put
you back at the normal command line. "mclog" will display the server
logs, so you can read through.

Make It Public
--------------

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

Alternate: Run on EC2
---------------------

A commenter on reddit mentioned that running this on EC2 may be a more
economical approach. If you don't have an EC2 account already, you can
[get some of the resources for free](http://aws.amazon.com/free) for a
year by using a tiny instance. I haven't tested this yet, and I am
curious how the performance would be, since the CPU for a tiny instance
is smaller than the others and burstable. Also, you will be charged for
the bandwidth over 15GB in or out, however I don't think many people
will likely hit that amount in a month. Plus, this way frees you up from
paying for the power, and keeps your home internet speedy.

Conclusion
----------

I hope this worked for you. If it didn't, post a note in the comments
and I will try to help you. If it did, post that too!! I like knowing
that I'm helping. If you have suggestions, please post those so everyone
can run this awesome game better. I'll be writing up another article in
a few days (hopefully) about some more advanced ways to manage your
server, including posting messages and server status to Facebook/Twitter
automagically.
