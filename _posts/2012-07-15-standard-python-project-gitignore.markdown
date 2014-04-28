---
title: Standard Python Project .gitignore


date: 2012-07-15 00:07

author: Josh

layout: default
category: Articles

tags: Git, Python

permalink: standard-python-project-gitignore
---

I write a lot of Python projects and store them all in Git version
control. Each time, I find myself rewriting the .gitignore file, so my
repositories aren't cluttered with .pyc files, temporary files, files
from pip/virtualenv, and others. So here is my current base .gitignore,
partly so it can help you, the reader, and partly so I know where it is.
I'll update it as I find other troublesome files. What do your
.gitignores look like?

{% highlight bash %}
# Python compiled files
\*.py[co1]
{% endhighlight %}

\# Packages \*.egg \*.egg-info dist build eggs parts bin var sdist
develop-eggs .installed.cfg

\# Installer logs pip-log.txt

\# Unit test / coverage reports .coverage .tox

\#Translations \*.mo

\#Mr Developer .mr.developer.cfg

\# Logging \*output.log

\#Kate editor \*kate-swp\*

\#Ubuntu One (if also stored for syncing) \*u1conflict\*

\#Mercurial \*/.hg/

\#Additional editor \*\~ \*.save

Post your .gitignore additions in the comments!
