---
title: How To Write Better Bash Scripts

date: 2011-06-07 20:14

author: Josh

layout: default
category: Articles

tags: BASH

permalink:  bash-how-to-write-better-scripts
---
If you haven't read my article on [Automated Installer for
Nginx](http://www.servercobra.com/automated-wordpress-install-with-nginx-script/),
you should (not to toot my own horn). It is a script containing around
200 lines of code, including the templates. I wrote it in about 2 hours,
mostly, because I had a great environment for testing. I'm going to
share my story and my clean up script, so that hopefully, it helps you
while you write your scripts.

Test Often
----------

I find BASH's syntax to be aggravating, to say the least. However, there
are a few ways to make life a bit easier. The most obvious is to use an
editor that supports BASH syntax, so it can highlight as you write your
code. I prefer nano. It helps point out things like unterminated
quotation marks.

Once you a few lines written, you should test for syntatical errors. It
is very simple! Just type:

{% highlight bash %}
bash -n scriptname.sh
{% endhighlight %}

Wasn't that easy? I suggest doing it for every 10 lines or so. I've been
writing BASH scripts for a couple years and still find myself making
minor errors here and there. It is much easier to catch them as you
write, rather than at the end. I've certainly had days where I spent
over an hour hunting a minor syntactical error.

Exit on Error
-------------

If you have a long script, where each part relies on the previous part
working, there is no reason not to add "set -e" at the top. This will
exit if any statement returns anything other than 0 (BASH for completed
successfully or true). Basically, if a single command has an error, it
will quit the whole script, rather than leaving you with a half working
system. This is especially important for automation scripts, where a
broken script can break the whole system. Another (possibly better)
option is to leave off "set -e", and do sanity checking after each
procedure. For example, if you create a configuration file for a daemon,
then restart the daemon, you could kill the daemon. In this instance, it
is probably better to revert to the previous configuration file, and
re-restart the daemon, so it is still running. I'd almost require this
for a production script. An example of "set -e" would be:

{% highlight bash %}
#!/bin/bash
{% endhighlight %}

set -e

....

Check All Variables Are Set
---------------------------

Much like "set -e", "set -u", or "no unset" is a great thing to prevent
your scripts from exiting for silly reasons. This simply checks to make
sure all variables are set before they are used, and exits if they
aren't. It sometimes is quite hard to debug a problem where one variable
just shows up as a 0 length string. This works great for production
scripts, and I'd say it is almost required. Here's how you do it!

{% highlight bash %}
#!/bin/bash
{% endhighlight %}

set -eu

....

Know Which Commands Are Being Executed
--------------------------------------

A great way to flip on debugging and see exactly which command is
throwing that annoying error. If you add "set -v", it will print out the
command it is executing right before it executes. This is handy for
shorter scripts and longer scripts alike, however, longer scripts can
get a bit tiresome. Obviously, don't leave this on for production! Let's
see all three together:

{% highlight bash %}
#!/bin/bash
{% endhighlight %}

set -euv

....

Cleanup Scripts Save Time and Headaches
---------------------------------------

If your script is the kind that builds a lot of configuration files or
does stuff with databases, or anything like that, a clean up script will
prevent you from boring yourself to death every time you script does
something other than expected. Basically, you want to undo what you just
did. Delete new files, reload daemons again, drop created database
users, etc. Here's an example I used for the nginx script:

{% highlight bash %}
#!/bin/bash
{% endhighlight %}

\# We won't use set -eu, because this is likely to error our if the
error in the script happened early.

\# Remove created files rm -rf /ebs/www/\$1/ rm /etc/php5/fpm/pools/\$2
rm /etc/nginx/sites-available/\$2 rm /etc/nginx/sites-enabled/\$2

\# Reload daemons so deletions take effect /etc/init.d/nginx reload
/etc/init.d/php5-fpm reload

\# Remove created user deluser \$1

\# Remove created MySQL database and user echo "drop database \$1; drop
user <$1@localhost;>" \> clear.sql mysql -u root -pPASSWORD \< clear.sql
rm clear.sql

[/bash]

Just like the script, this takes a username and a domain name for its
arguments. Similar scripts will make your life much, much easier.

Have any good BASH script tutorials? Post them in the comments!
