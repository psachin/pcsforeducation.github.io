---
title: Installing Minecraft On Ubuntu


date: 2011-11-02 13:51

author: Josh

category: Articles

tags: Minecraft, Ubuntu

permalink: installing-minecraft-on-ubuntu
---

Minecraft has become one of the most popular games among geeks. Unlike
many games, it runs fantastically (actually better in my case) than on
Windows! However, the installation process is a bit more involved. This
guide will help you get it installed and running smoothly.

**Ubuntu Versions:**Tested on 10.04 through 11.10, 32 and 64 bit.

**Installation:** Installation of Minecraft is surprisingly easy. First,
we need to get the official version of Java from Oracle, then we
download a Java .jar file, and run it!

Ubuntu comes with the OpenJDK, which doesn't work well with Minecraft.
Let's download the official version, and set that as the default for
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
# Select 2 here. It should be something like "/usr/lib/jvm/java-6-sun/jre/bin/java"
{% endhighlight %}

Now that we have Java installed, we can get the client. You can either
go to Minecraft.net and download it, or you can just download it from
the command line. We've already got the command line open, so let's use
that. We're going to put it in the games folder so other users can
easily access Minecraft also.

{% highlight bash %}
sudo mkdir /usr/local/games/minecraft
sudo wget -O /usr/local/games/minecraft/minecraft.jar https://s3.amazonaws.com/MinecraftDownload/launcher/minecraft.jar
# Make it executable, for good measure
chmod 755 /usr/local/games/minecraft/minecraft.jar
{% endhighlight %}

Now that we have the game files downloaded and in place, we need to run
it to finish the install by simply starting the game! We're going to
make an alias in bash to start Minecraft up. If you want to make a
desktop shortcut, simply copy the part in quotations into the launcher.

{% highlight bash %}
nano ~/.bash_aliases

--
alias minecraft="java -Xmx1024M -Xms512M -cp
/usr/local/games/minecraft/minecraft.jar net.minecraft.LauncherFrame &"
--

# Reload terminal for alias to take effect
bash
{% endhighlight %}

Now to launch the game, all you have to do is type:

{% highlight bash %}
minecraft
{% endhighlight %}

Log in with your account and it will download the rest of the files and
place them in \~/.minecraft.

One common problem people have with playing Minecraft on Ubuntu is the
infamous stuck-key problem. Basically, you'll be going in one direction,
hold down a key, and when you release the key, you'll keep going. This
is a serious problem if you're headed towards a pool of lava and the
game keeps you moving forward to your pixely and block-filled doom, or
building a tall structure, only to be fall off and have your blocky
items explode all over the ground. To fix this, we just need to update a
few files. There is a handy script to do this for you.

Note: only use this after running Minecraft once (so it installs all the
required files).

{% highlight bash %}
cd /tmp
wget -O lwjgl.tar.gz https://files.dafrito.com/upgrade-lwjgl.tar.gz
tar xvf lwjgl.tar.gz
sudo ./upgrade-lwjgl.sh
{% endhighlight %}

Have fun playing Minecraft!!

If you want to install a Minecraft server so you can play with your
friends, check the [Minecraft Server On
Ubuntu](http://www.servercobra.com/minecraft-server-on-ubuntu/) article.
If you'd like to play a pre-release version of Minecraft, which has all
the latest updates and additions but a few more bugs, check the
[Minecraft 1.0 Ubuntu
Installation](http://www.servercobra.com/minecraft-1-0-ubuntu-installation/)
article.

If anything didn't work for you, please leave a comment and I'll try and
help you! If it did work, I always appreciate a quick "It worked!"
comment!
