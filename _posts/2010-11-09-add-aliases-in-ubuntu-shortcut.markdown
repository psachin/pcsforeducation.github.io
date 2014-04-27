---
title: Add Aliases in Ubuntu Shortcut

category: Articles

tags: Scripts, Time Savers, Ubuntu

permalink:  add-aliases-in-ubuntu-shortcut
---
I love the Ubuntu terminal. I spend more time in it than any sane person
should. So obviously, I customize it as much as possible to make it work
exactly how I want. The easiest way I've found is through aliases. An
example of an alias is when I type "update" into my terminal, the real
command it runs is "sudo apt-get -y update && sudo apt-get -y upgrade"
(Automagically updates and upgrades all the packages on my system).
While searching for other fun shortcuts, I came across [this helpful
post](http://brockangelo.com/2009/05/30/my-top-10-ubuntu-aliases/). It
provides a quick way to install aliases into your system. I've modified
it to make it a bit better (I think). All credit for the idea still goes
to Brock Angelo, the original author.

Fire up your favorite editor, and save this file either in you home
directory (/home/username/aliasgen.sh) or I prefer to save it into
/bin/aliasgen (requires sudo) so all users can access it. We'll set up a
handy alias for it also, so it doesn't really matter. Here's the
modified code for the script:

{% highlight bash %}
#!/bin/bash
echo Enter the shortcut, or alias, you want to use:
read SHORTTEXT
echo Now enter what text you want it to replace:
read LONGTEXT
echo "alias $SHORTTEXT='$LONGTEXT'" >> ~/.bash_aliases
echo "Alias $SHORTTEXT='$LONGTEXT' Type 'bash' for changes to take effect."
{% endhighlight %}

The main change I made was adding the text to \~/.bash_aliases instead
of the main \~/.bashrc. My \~/.bashrc has other customizations (which
I'll get to in a future post), and the standard Ubuntu bashrc already
has support for this file to keep everything organized. Also, I added
the handy point at the end, that you only need to type 'bash' into each
terminal you have open currently for it to take immediate effect (I
don't log out for months at a time).

Anyways, let's alias this! An alias for a program to make aliases to
another program is getting a bit Inception-like, but oh well. Our alias
will run the program, then immediately restart the terminal so changes
take effect immediately. Head to the terminal and run the program:

{% highlight bash %}
#If you saved it in /bin, first change permissions (first time only), then run it
sudo chmod 755 /bin/aliasgen
/bin/aliasgen

#Or in your home directory
./aliasgen.sh

#We can't use 'alias', as that is a Linux command, which we can't override
#Instead input something like:
al
#Then
/bin/aliasgen; bash;
#Or, if you saved it in your home directory
bash ~/aliasgen.sh; bash;
{% endhighlight %}

That should about cover it! Run it an see how you like it!
