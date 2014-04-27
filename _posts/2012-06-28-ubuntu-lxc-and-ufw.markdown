---
title: Ubuntu LXC and UFW


date: 2012-06-28 18:00

author: Josh

category: Articles

tags: LXC, Ubuntu, UFW

permalink: ubuntu-lxc-and-ufw
---

I am working on setting up my own cloud servers using Linux Containers
(LXC). These cloud containers allow me to build one image, replicated
it, configure it, and have a new server online in seconds.

I was having trouble getting access to the Internet from containers
whenever I enabled UFW on the host machine. I tried a thousand different
rules, but none worked. After much digging, I found the solution. UFW
defaults to dropping forwarded packets. All my containers interact with
the Internet via forwarding over a bridge (lxcbr0 in 12.04), so this is
a big problem. Basically, with UFW enabled, the LXC containers could not
talk to the internet. To solve the problem:

{% highlight bash %}
sudo nano /etc/default/ufw
----
# Change:
# DEFAULT_FORWARD_POLICY="DROP"
# to
DEFAULT_FORWARD_POLICY="ACCEPT"
{% endhighlight %}

Once I changed the rules, I just reload UFW, and all was well.

{% highlight bash %}
ufw reload
{% endhighlight %}
