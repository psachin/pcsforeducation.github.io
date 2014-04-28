---
title: Fabric XMPP Chat
date: 2013-08-09 16:02
layout: default
category: Articles
tags: Python Fabric
permalink: frabric-xmpp-chat
---
In an effort to streamline my own deployment system, I decided to make
Fabric (and soon, the rest of my infrastructure) send me chat messages
when things happen. This is a very simple version of the system to get
you started. Basically, you have it connect to an XMPP server (I'm using
ejabberd on Ubuntu 12.04), and then send you messages on certain events.
This one will shoot out a message when you start the deployment, and
when it completes. Handy additions would be an error handler of some
sort, which relays those messages to you. This system will be very
useful with a continuous integration server (which I'm working on right
now), so it can tell you what its progress is, and you can send it
queries and commands via a simple chat interface. This whole thing was
inspired by Github's HUBOT.

<!--more-->

Make sure you have Fabric and the xmpp library installed (preferably in
your virtualenv):

{% highlight python %}
pip install fabric xmpppy
{% endhighlight %}

Now we will add a few handy functions to our fabfile.py:

{% highlight python %}
env.xmpp_auth = {}
env.xmpp_client = None

def configure_xmpp(config_file='/etc/xmpp_credentials.ini', section=None):
    config = ConfigParser.ConfigParser()
    config.read(config_file)
    if section is None:
        section = env.project
    if not config.has_section(section):
        print "Could not find section {0} in config file: {1}".format(section, config_file)
        return

    env.xmpp_auth['username'] = config.get(section, 'username')
    env.xmpp_auth['password'] = config.get(section, 'password')
    env.xmpp_auth['hostname'] = config.get(section, 'hostname')
    env.xmpp_auth['to'] = config.get(section, 'to')
    print env.xmpp_auth

def send_xmpp_message(msg):
    if env.xmpp_client is None:
        env.xmpp_client = xmpp.Client(env.xmpp_auth['hostname'], debug=[])
        env.xmpp_client.connect()
        env.xmpp_client.auth(env.xmpp_auth['username'], env.xmpp_auth['password'])
        env.xmpp_client.sendInitPresence()
    message = xmpp.Message(env.xmpp_auth['to'], msg)
    message.setAttr('type', 'chat')
    env.xmpp_client.send(message)
{% endhighlight %}

Before you do anything, you should call configure_xmpp(). This will
save the credentials from you credentials file in your env variable for
use in send_xmpp_message(). Then, each time you want to send a message
to yourself, call send_xmpp_message, and bam! You get it.

Finally, we need to create a credentials file. This should not be stored
with your code (so it doesn't wind up in version control). The code
above defaults to /etc/xmpp_credentials.ini.

{% highlight bash %}
[scouter]
username=scouter
password=SECURE_PASSWORD
hostname=servercobra.com
to=user@servercobra.com
{% endhighlight %}
