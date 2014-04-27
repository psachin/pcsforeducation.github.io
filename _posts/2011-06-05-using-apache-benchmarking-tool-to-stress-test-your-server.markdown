---
title: Using Apache Benchmarking Tool to Stress Test Your Server


date: 2011-06-05 12:08

author: Josh

category: Articles

tags: Ubuntu

permalink: using-apache-benchmarking-tool-to-stress-test-your-server
---

So you've followed my previous tutorials on how to set up a server using
nginx, add in caching for better performance, and then how to automate
the creation of new sites. However, with a new server, how can you tell
how much of a load you can really deal with? Now, since I'm using a tiny
instance on Amazon EC2, I don't expect performance to be amazing.
However, I'm already getting much better performance than when I was
using Apache. One of many answers is to using Apache (ironic, right?)
Benchmarking Tool, or "ab". Let's dig in and get some real statistics.
Before we start, consider what you are doing! You are going to try and
push your server as hard as you possibly can. With my old Apache server,
I was easily able to crash my server. Do this at your own risk! Probably
a good idea to take an early nap, wake up during your lowest load time,
and do it then. Another option, especially with EC2, is you can clone
the server and use that one for load testing. I'd highly recommend this
method! You're going to want to fire up to different terminals first.
Log into your web server via SSH on one, and on the local one, install
ab.

{% highlight bash %}
sudo apt-get -y -q install apache-utils
{% endhighlight %}

On your server, you will want to check a few things. The tool "free"
will list how much memory is free/used, and how your CPU usage is.

{% highlight bash %}
# To check free memory:
free -m

# To check your CPU usage (and many, many other stats), check top. CPU usage is in the upper left corner.
top
{% endhighlight %}

The used column lists how much memory is actually used, which looks high
on the top row. The second row is memory - memory used for caching. This
is how much your programs are actually using. The caching is caching
done via the OS, not the previously mentioned methods. I've got a few
sites on my server, so hopefully your usage is a bit lower. Also, some
of that is used by APC for memcaching. The bottom row should hopefully
have basically 0 used at all times, or you need to do some tuning or
consider getting a bigger server. For top, I'd recommend just reading
through "man top". The stat we are concerned most about is in the
"CPU(s)" row, and the first statistic, which looks like "3.6%us". We'd
like to see this number stay relatively low, but on a tiny instance,
where you basically have burstable CPU...that's a pipe dream. When you
are running our tests, you are going to want to keep a window open and
check these statistics, just to see how your server deals with it. This
can give you ideas on what you need to do to get better results. So
let's dig in on ab. Unfortunately, I am on a terribly slow connection,
so I'll add my own servers statistics later.

{% highlight bash %}
# For our first test, something very quick and simple
ab -c 1 -n 10 http://example.com/

# A few notes. -c is concurrent hits, -n is requests per -c, and the last is the page you want to hit. Remember the trailing slash! This is a pretty easy test. Let's crank it up!

ab -c 10 -n 1000 http://example.com/

# That should be a decent test (and will take some time). If it completes with 99% under 1000ms, that's pretty good.
# Now let's tweak a few things to put a more realistic load on it.

ab -c 10 -n 1000 -k http://example.com/

# This will do the same test, except it will keep connections alive (-k). This should give you a bit different results.
# Did you survive?
{% endhighlight %}

I have a few improvements that might make this a bit better, including
using another server in a different datacenter and using gnuplot to
automatically display the load over time.
