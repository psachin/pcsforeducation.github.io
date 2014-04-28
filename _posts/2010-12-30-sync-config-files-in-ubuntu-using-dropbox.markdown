---
title: Sync Config Files in Ubuntu using Dropbox


date: 2010-12-30 00:37

author: Josh

layout: default
category: Articles

tags: Ubuntu

permalink: sync-config-files-in-ubuntu-using-dropbox
---

Dropbox is a fantastically amazing program to keep documents synced and
backed up across computers. It also works wonders for syncing config
files! I'm going to show you how to sync your \~/.bashrc file between
computers.  I use this one to add a bunch of aliases and other things to
make my command line life a bit easier.

First grab the [Dropbox Ubuntu Installer (32
bit)](http://www.dropbox.com/download?dl=packages/nautilus-dropbox_0.6.7_i386.deb&src=index)
or the [Dropbox Ubuntu Installer (64
bit)](http://www.dropbox.com/download?dl=packages/nautilus-dropbox_0.6.7_amd64.deb&src=index).
You can also head over to the [Dropbox download
page](http://www.dropbox.com/downloading?src=index) if the links get
stale or outdated. Double click the installer, and it will open the
Ubuntu Software Center. Simply hit Install, and it will complete most of
the process for you.

The GUI installer will create a folder called \~/Dropbox. Then go to the
terminal again.

We are first going to make a new folder in Dropbox called config. Then
we are going to move our bashrc file into it, then make a symbolic link
where the file used to be.

{% highlight bash %}
cd ~/Dropbox
mkdir config
mv ~/.bashrc ~/Dropbox/config
ln -s ~/Drobox/config/.bashrc ~/.bashrc
{% endhighlight %}

Now personally, I don't want to have to do this on each computer I am
syncing, so in a very SysAdmin style, let's script it.

{% highlight bash %}
mkdir ~/Dropbox/scripts
nano ~/Dropbox/scripts/sync_config.sh
mv ~/.bashrc ~/Dropbox/config/bashrc
ln -s ~/Dropbox/config/bashrc ~/.bashrc
{% endhighlight %}

\#!/bin/bash

\#Repeat for each config file you want synced rm \~/.bashrc ln -s
\~/Dropbox/config/bashrc \~/.bashrc

Now whenever you get a new computer that needs the config files synced,
simply install Dropbox, then run the script:

{% highlight bash %}
sh ~/Dropbox/scripts/sync_config.sh
{% endhighlight %}

Credit for this one goes to
<http://www.nixtutor.com/linux/sync-config-files-across-multiple-computers-with-dropbox/>
