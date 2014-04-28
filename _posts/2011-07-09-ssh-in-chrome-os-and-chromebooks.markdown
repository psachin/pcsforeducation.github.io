---
title: SSH in Chrome OS and Chromebooks


date: 2011-07-09 09:55

author: Josh

layout: default
category: Articles

tags: chrome os, Chromebook

permalink: ssh-in-chrome-os-and-chromebooks
---

One problem a lot of people seem to have is using the odd SSH client on
Chromebooks and in Chrome OS. Chrome OS uses a shell called crosh, which
may stand for "Crippled, read only shell". Here's how to use SSH to
connect to servers.

Assuming you are not in developer mode, to get to the Chrome OS shell,
hit Ctrl + Alt + T. Unlike other shells, crosh has its own SSH
sub-shell. Simply type **ssh**, and you'll be in the SSH sub-shell.

For a quick example, let's connect to example.com on port 22222 as user
servercobra, using a key called Key.pem stored in the download
directory.

{% highlight bash %}
ssh
host example.com
port 22222
user servercobra
key Key.pem
connect
{% endhighlight %}

\# And when you're done and exit the SSH session: quit

\# To exit the crosh shell exit

So you need to specify all of your options on their own lines, unlike
the standard SSH client on Linux. Once you have your options specified,
just type "connect". Type "help" for the other options, such as dynamic
port forwarding and standard port forwarding.

If you are in developer mode, you can simply hit Ctrl + Alt + T, then
write "shell", to get to a standard Bash shell, or Ctrl + Alt + -\> to
get to that shell as well. From there, you can use SSH like normal.
