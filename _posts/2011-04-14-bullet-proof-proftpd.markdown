---
Title: Bullet Proof ProFTPD

date: 2011-04-14 22:45

author: Josh

category: Articles

tags: FTP, Ubuntu, Webserver

permalink:  bullet-proof-proftpd
---
Ever needed an FTP server? Need it to be easy to work with? Look no
farther than ProFTPD on Ubuntu! Here's how to install it, then secure
it. If the tutorial works or doesn't work, please leave a comment at the
end, so I can improve my tutorials.

**Platform**: Ubuntu 10.04+ (Should work for older versions)

This tutorial assumes you will be putting user websites in their home
directories. You should be able to do put websites in /var/www/username
if you really want, but you will have to change the configuration some.

First, the install. We're going to make this silent. I realize this
really serves no purpose if you're installing from commandline, but
these hints will help you (and me!) write a script.

{% highlight bash %}
sudo apt-get -y install debconf-utils
echo "proftpd-basic shared/proftpd/inetd_or_standalone select standalone" \| debconf-set-selections
sudo apt-get -y install proftpd
{% endhighlight %}

Now, we are going to add a proftpd group. Any user that wants to use FTP
will just be added to this group (and will be able to FTP to their home
directory).

{% highlight bash %}
groupadd proftpd
{% endhighlight %}

Now, most of the security and configuration is in the configuration
file:

{% highlight bash %}
#Open config file
sudo nano /etc/proftpd/proftpd.conf
-----
#Name showed to connecting users
ServerName "Ubuntu FTP"
#Your admin email
ServerAdmin admin@example.com
#Jail users in their home directory
DefaultRoot ~
#For security purposes, look up the host name of users as they connect
IdentLookups on

#TimeoutNoTranser, TimeoutStalled, and TimeoutIdle should all be set automatically.
#If not, set them to appropriate values (in seconds), other than 0

#Add TimeoutLogin for added security
TimeoutLogin 120
#Change group to proftpd to facilitate easy FTP user adding
Group proftpd
#Allow file overwriting of files in all directories
<Directory /\*>
AllowOverwrite on
</Directory>
#Now we're going to prevent users from creating files outside of their home directories.
<Limit WRITE>
DenyAll
</Limit>

#No security solution is worthwhile without logging. We need a way to figure out what happened when someones inevitably breaks in
#You probably already have TransferLog defined elsewhere. If so, delete it.
TransferLog /var/log/proftpd/xferlog.default
LogFormat default "%h %l %u %t \\"%r\\" %s %b"
LogFormat auth "%v [%P] %h %t \\"%r\\" %s"
LogFormat write "%h %l %u %t \\"%r\\" %s %b"
UseReverseDNS off
ExtendedLog /var/log/proftpd/access.log WRITE,READ write
ExtendedLog /var/log/proftpd/auth.log AUTH auth
ExtendedLog /var/log/proftpd/paranoid.log ALL default
{% endhighlight %}

Now we have this super duper secure server right? WRONG! The big thing
we are missing is encryption. Right now, as users log in, their username
and password are sent in plaintext, meaning if anyone intercepted it,
they could easily read your username and password, login, and do
whatever they want.

{% highlight bash %}
sudo nano /etc/proftpd/modules.conf
-----
#Ensure your TLS Module is enabled like this:
#LoadModule mod_tls.c

#Add config details
<IfModule mod_tls.c>
    TLSEngine on
    TLSLog /var/ftpd/tls.log

    # Support both SSLv3 and TLSv1
    TLSProtocol SSLv3 TLSv1

    # Are clients required to use FTP over TLS when talking to this server?
    TLSRequired off

    # Server's certificate
    TLSRSACertificateFile /etc/proftpd/server.cert.pem
    TLSRSACertificateKeyFile /etc/proftpd/server.key.pem

    # Authenticate clients that want to use FTP over TLS?
    TLSVerifyClient off

    # Allow SSL/TLS renegotiations when the client requests them, but
    # do not force the renegotations. Some clients do not support
    # SSL/TLS renegotiations; when mod_tls forces a renegotiation, these
    # clients will close the data connection, or there will be a timeout
    # on an idle data connection.
    TLSRenegotiate none

</IfModule>
{% endhighlight %}

Now we just need to get an SSL certificate. This will pop up some
warning messages, but it beats paying hundreds of dollars a year to
Verisign.

{% highlight bash %}
sudo apt-get -y install openssl
openssl req -new -x509 -days 365 -nodes -out /etc/proftpd/server.cert.pem -keyout /etc/proftpd/server.key.pem
{% endhighlight %}

Now just fill in the information that it requests, and restart the
server:

{% highlight bash %}
sudo /etc/init.d/proftpd restart
{% endhighlight %}

To add a user to the FTP server, simply run:

{% highlight bash %}
usermod -g proftpd username
{% endhighlight %}

That should cover everything! Did it work for you? Did you have issues?
Either way, leave a comment! For more server-tastic articles, [subscribe
to my RSS feed](http://servercobra.com/feed)!

Credit: Sweet silent install:
[http://wiki.mediatemple.net/w/(ve):Install\\_ProFTPD\\_on\\_Ubuntu](http://wiki.mediatemple.net/w/(ve):Install_ProFTPD_on_Ubuntu)
Security:
<http://www.techrepublic.com/article/lock-it-down-set-up-a-secure-ftp-server-with-proftpd/5031101>
Security:
<http://www.ubuntugeek.com/settingup-an-ftp-server-on-ubuntu-with-proftpd.html>
