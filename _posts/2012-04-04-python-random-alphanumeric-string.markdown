---
title: Random Alphanumeric String in Python


date: 2012-04-04 15:47

author: Josh

layout: default
category: Articles

tags: Python

permalink: python-random-alphanumeric-string
---

I find I need to generate a random alphanumeric string quite often,
whether for a password, username, or even file names. This handy
function takes a length argument for how many characters you want. If
you put in less than 1, it will simply return an empty string.

{% highlight python %}
import random
import string

# Returns a random alphanumeric string of length 'length'
def random_key(length):
    key = ''
    for i in range(length):
        key += random.choice(string.lowercase + string.uppercase + string.digits)
    return key
{% endhighlight %}
