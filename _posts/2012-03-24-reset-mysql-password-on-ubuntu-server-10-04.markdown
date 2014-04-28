---
title: Reset MySQL Password on Ubuntu Server 10.04


date: 2012-03-24 23:23

author: Josh

layout: default
category: Articles

tags: MySQL, Ubuntu

permalink: reset-mysql-password-on-ubuntu-server-10-04
---

Sometimes, shit happens. If you're reading this, it probably just
happened. Either you forgot your password or somehow you set it to
something else through a script, or something else equally terrible
happened. Regardless, you need to reset your password. First, you should
probably make a backup of your data in case something goes horribly
wrong. This should work on almost any version of Ubuntu (or other
versions of Linux), but it is tested on Ubuntu 10.04 with mysql-server
5.1.

First, we need to shut down MySQL. This is going to bring all your
database-driven sites down (though caching may save you). Then we will
restart it, while ignoring the permissions table.

{% highlight bash %}
sudo stop mysql
sudo mysqld --skip-grant-tables &
# Note the PID that is printed out, such as "[1] 14053", where "14053" is the PID.
{% endhighlight %}

Now we log in with no password, and reset the password.

{% highlight bash %}
mysql -u root
UPDATE user SET Password=PASSWORD('YOURNEWPASSWORD') WHERE User='root'; FLUSH PRIVILEGES; exit;
{% endhighlight %}

The password is reset! Now we need to kill off the passwordless
instance, and restart a normal MySQL server.

{% highlight bash %}
# PID is the number from before, probably 4 or 5 digits
sudo kill PID
sudo start mysql
{% endhighlight %}

Now make sure you can log in!

{% highlight bash %}
mysql -u root -p YOURNEWPASSWORD
{% endhighlight %}
