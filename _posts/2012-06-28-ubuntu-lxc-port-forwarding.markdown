---
title: Ubuntu LXC Port Forwarding


date: 2012-06-28 20:37

author: Josh

layout: default
category: Articles

tags: LXC, Ubuntu

permalink: ubuntu-lxc-port-forwarding
---

So you have a service running in an LXC container behind a bridge, and
you need ports forwarded for that service. In this case, we'll use
Minecraft as an example, which runs on port 25565. Our container's IP is
10.0.3.116.

{% highlight bash %}
sudo iptables -t nat -I PREROUTING -p tcp -d $PUBLICIP --dport 25565 -j DNAT --to 10.0.3.116:25565
sudo iptables -A FORWARD -p tcp -d 10.0.3.116 --dport 25565 -j ACCEPT
{% endhighlight %}
