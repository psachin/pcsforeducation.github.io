---
title: Check If Ubuntu Package Installed Using Bash

date: 2010-11-08 20:52

author: Josh

category: Articles

tags: BASH

permalink:  bash-check-if-package-is-installed
---
**Platform(s)**: Ubuntu (probably Debian too) Sometimes you need to take
an action based on whether or not a particular package (first argument)
is installed or not. This script checks if the package is installed,
then takes action based on that. If it's not installed, the script
exits.

**Update:**As Reef pointed out in the comments, if you are searching for
a package name with grep, you could get other packages. The example
given was checking if the package "zip" is installed, which may not, but
packages like "bzip" are, you'll get a false positive. Rather, he
provided code which uses the exit status code of dpkg to determine
success or failure. Specifically, if dpkg doe have a package by that
name, it will return 0. If it does not, it will return 1. We can check
the exit status code with "\$?". Therefore, the improved code is:

{% highlight bash %}
#!/bin/bash
dpkg -l $1 >/dev/null 2>&1 && echo -n “not ” ; echo “installed”

# Alternatively, in a much longer, and more roundabout fashion:
dpkg -l $1 > /dev/null 2>&1
INSTALLED=$?

if [ $INSTALLED == '0' ]; then
    echo "installed"
else
    echo "not installed"
fi
{% endhighlight %}

{% highlight bash %}
#!/bin/bash
INSTALLED=$(dpkg -l \grep $1)
if [ "$INSTALLED" != "" ]; then
    echo "installed"
else
    echo "not installed"
    exit
fi
{% endhighlight %}

Run the command like this:

{% highlight bash %}
bash your-script-name.sh package
{% endhighlight %}
