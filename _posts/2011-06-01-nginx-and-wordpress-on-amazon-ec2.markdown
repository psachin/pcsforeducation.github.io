---
title: Nginx and Wordpress on Amazon EC2


date: 2011-06-01 08:12

author: Josh

category: Articles

tags: EC2, nginx, Webserver, Wordpress

permalink: nginx-and-wordpress-on-amazon-ec2
---

I'm bored with Apache. And it is slowing down my server (though part of
that is probably my lack of awesome tuning abilities). It feels too much
like old software that is in dire need of being rewritten for today's
server requirements and I don't like that it requires a lot of tuning to
get it to work well in small server situations. It certainly has a place
(I'm told), but so far, I haven't figured out where that is. Both my
Django projects and Wordpress sites are easily migratable to nginx
(officially pronounced "engine x", but I can't stop saying "N jinx"),
and the performance I've seen posted on the web is astonishing. See
[this blog post](http://bit.ly/itbExA), forwarded to me via @WurvHosting
for a great script to do something quite similar.

**Update:**Updating this to move more configuration files to the EBS
volume, and XFS allows us to do consistent snapshotting more easily.
This will help us on the next article, where we learn how to make hourly
backups.

First, we need to spin up an EC2 instance. Head over to [the Amazon EC2
Console](http://aws.amazon.com/console). Hit "Launch Instance", and find
the Ubuntu image with AMI ID "ami-3e02f257". **Update:** It seems the
more current one is "ami-61be7908". Please check the [Ubuntu AMI
listing](http://cloud.ubuntu.com/ami/) for the latest image. For this
article, we'll use the latest US East, Lucid 32-bit, EBS storage AMI.
Pick whatever settings you want, though I recommend a tiny image (free
if you're new to Amazon AWS for 1 year), and a security setup with only
ports 22, 80, and 443 open (SSH, web, and secure web, respectively).
After that, create a new EBS volume. I choose to make it 5GB, because it
is only \$0.50 a month, and should be enough storage. If later you need
to make it large, simply snapshot the volume and clone it to a larger
size from the AWS Console. Attach the new EBS volume to the running
instance at "/dev/sdf".

**Note:**All commands assume you are running as root. To change from the
default user, ubuntu, simply type "sudo su".

{% highlight bash %}
fdisk /dev/sdf
--
n (new)
p (primary partition)
1 (1st partition)
1 (1st cylinder)
enter (last cylinder)
w (write partition table to disk)
--
#Install the tool to format using the XFS filesystem
apt-get -y -q install xfsprogs
#Format the new partition
mkfs.xfs /dev/sdf1
#Mount the newly formatted partition
mkdir /ebs
mount /dev/sdf1 /ebs
{% endhighlight %}

Now let's get the software we need installed, part by part. We are going
to copy all the important files to /ebs and then symlink them back to
the folder. This way we can easily snapshot the important files in one
go, making nightly backups trivial. We can also make those snapshots
available around the world, so if an entire Amazon datacenter goes down
(which happened recently), we can clone the snapshot and get a temporary
server running elsewhere in a short amount of time. We won't cover that
here (yet..), but it is very doable and recommended. Let's get
everything updated first:

{% highlight bash %}
apt-get -q -y update && apt-get -q -y upgrade
{% endhighlight %}

Now we are going to install nginx. Nginx is a great alternative to
Apache and has most of the features we need built in (user separation,
etc). We are going to use a PPA, since the version in the Ubuntu mirrors
is really old.

{% highlight bash %}
apt-get -q -y install python-software-properties
apt-get -q -y update
add-apt-repository ppa:nginx/stable
apt-get update && apt-get install nginx
{% endhighlight %}

The only modification we really need to make to the nginx configuration
file is the number of worker processes. We will use the number of EC2
virtual cores our instance has, which represents how many cores our
server can access. For a tiny instance, that number is simply one. For
the full listing, see [this AWS EC2
document](http://aws.amazon.com/ec2/instance-types/). Change
"worker_processes 1" to the number of virtual cores your instance type
has.

{% highlight bash %}
nano /etc/nginx/nginx.conf
--
# Change "worker_processes NUM;" to:
worker_processes 1;
# Make sure these lines are uncommented:
gzip on;
gzip_disable "msie6";
gzip_proxied any;
gzip_comp_level 3; #You can vary this. 1 is least compression, 9 is most. I'll keep it low, since we have low CPU power.
gzip_buffers 16 8k;
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
--
# Now we are going to migrate these files to the EBS volume
/etc/init.d/nginx stop
mkdir /ebs/log /ebs/etc
mv /var/log/nginx /ebs/log/
mv /etc/nginx /ebs/etc/
mkdir /etc/nginx
mount --bind /ebs/etc/nginx /etc/nginx -o noatime
mkdir /var/log/nginx
mount --bind /ebs/log/nginx /var/log/nginx -o noatime
/etc/init.d/nginx start
{% endhighlight %}

Let's get MySQL installed. We are going to shrink it down some to take
up less RAM. Less RAM used by the database means more RAM for the
application!

{% highlight bash %}
DEBIAN_FRONTEND=noninteractive
apt-get -y -q install mysql-server mysql-client
{% endhighlight %}

Now let's make MySQL a bit more efficient. We are going to disable
InnoDB for extra performance. In situations where transactions need to
be absolutely guaranteed, InnoDB is awesome, but at a performance cost.
For us, if somehow our instance dies, it shouldn't be the end of the
world, and I suspect the way Amazon's EC2 and EBS are set up, this type
of loss may be even less likely than in a traditional server setup,
though I have no proof on hand to back that up.

**Update:** Added "default-storage-engine = myisam" to avoid MySQL not
starting.

{% highlight bash %}
nano /etc/mysql/my.cnf
--

# Under [msyqld] add
skip-innodb
default-storage-engine = myisam
--
{% endhighlight %}

Finally, we need to set up the root password for MySQL, and create a
user/password for our new Wordpress installation.

{% highlight bash %}
mysqladmin -u root password NEWPASSWORD

# Now create a new user and password, plus a database to store Wordpress's information in.
mysqladmin -u root -p create "example.com"
# You will then be prompted for your password

# Note: if "example" is longer than 15 characters, truncate it, as MySQL username cannot be longer than 15 characters
# Change example to your domain name minus '.com' or '.net' etc, and create a secure password for dbpassword
echo "GRANT ALL PRIVILEGES ON example.\* TO 'example'@localhost IDENTIFIED BY 'dbpassword';" \mysql -u root -p
{% endhighlight %}

Now we need to migrate the configuration files and database files over
to the EBS volume. Credit for this goes to [Eric
Hammond](http://aws.amazon.com/articles/1663?_encoding=UTF8&jiveRedirect=1).

{% highlight bash %}
/etc/init.d/mysql stop
mkdir /ebs/lib

# Move config files
mv /etc/mysql /ebs/etc/
mv /var/lib/mysql /ebs/lib/mysql
mv /var/log/mysql /ebs/log/mysql

# Mount the directories back into the filesystem
mkdir /etc/mysql
mount --bind /ebs/etc/mysql /etc/mysql -o noatime

mkdir /var/lib/mysql
mount --bind /ebs/lib/mysql /var/lib/mysql -o noatime

mkdir /var/log/mysql
mount --bind /ebs/log/mysql /var/log/mysql -o noatime

service mysql start
{% endhighlight %}

Let's get PHP installed, so we can actually run Wordpress. We are going
to install from source so we can use the more efficient PHP-FPM module
found in PHP 5.3.3. The default in the Ubuntu mirros is 5.3.2. So close!
So instead we will add another PPA.

{% highlight bash %}
add-apt-repository ppa:brianmercer/php && apt-get update
apt-get -q -y install php5-cli php5-common php5-mysql php5-suhosin php5-gd
apt-get -q -y install php5-fpm php5-cgi php-pear php5-memcache php-apc
service php5-fpm start

# Now we need to move our PHP configurations to EBS
/etc/init.d/php5-fpm stop
mv /etc/php5/fpm /ebs/etc/
mv /var/log/php5-fpm.log /ebs/log/php5-fpm.log

mkdir /etc/php5/fpm
mount --bind /ebs/etc/fpm /etc/php5/fpm -o noatime

#Since php5-fpm is a single file instead of a directory, we will use a symbolic link
ln -s /ebs/log/php5-fpm.log /var/log/php5-fpm.log

/etc/init.d/php5-fpm start
{% endhighlight %}

Now we need to configure PHP-FPM for optimal results. You can tweak all
of these settings to suit your needs, or to handle more traffic, etc.
This setup worked great for my little site, and left around 500MB of
free RAM, even under a decent load.

{% highlight bash %}
nano /etc/php5/fpm/php5-fpm.conf
--

# Change daemonize = no to:
daemonize = yes
# We will leave the default pool around, but create our own for security
# Do this per site for best security and tuning per site
# We will put each pool into it's own file, and include them all here.
include=/etc/php5/fpm/pools/\*
--
{% endhighlight %}

Now, since all these configuration files are not where they are supposed
to be, we will have problems on reboot. Therefore, we are going to mount
all these folders on boot up. Some have suggested putting them in
/etc/fstab, where most filesystem mounts are defined, but this can cause
your system to not boot if anything's wrong. It is just as easy to do in
a simple startup script.

{% highlight bash %}
# First let's disable startup of nginx and php5-fpm
update-rc.d -f mysql remove
update-rc.d -f php5-fpm remove
update-rc.d -f nginx remove

# Now we will make our script
mkdir /etc/servercobra
nano /etc/servercobra/S99ebs-mounts
----
# Script checks to see if EBS if attached at /dev/sdf, then mounts and starts services
if [-f /dev/sdf1 ]; then
mount --bind /ebs/etc/nginx /etc/nginx -o noatime
mount --bind /ebs/etc/mysql /etc/mysql -o noatime
mount --bind /ebs/etc/fpm /etc/php5/fpm -o noatime
mount --bind /ebs/lib/mysql /var/lib/mysql -o noatime
mount --bind /ebs/log/mysql /var/log/mysql -o noatime
mount --bind /ebs/log/nginx /var/log/nginx -o noatime
ln -s /ebs/log/php5-fpm.log /var/log/php5-fpm.log
service mysql start
/etc/init.d/php5-fpm start
/etc/init.d/nginx start
fi

--

# Lastly, we need to make this start up at various run levels.
ln -s /etc/servercobra/S99ebs-mounts /etc/rc2.d/S99ebs-mounts
ln -s /etc/servercobra/S99ebs-mounts /etc/rc3.d/S99ebs-mounts
ln -s /etc/servercobra/S99ebs-mounts /etc/rc4.d/S99ebs-mounts
ln -s /etc/servercobra/S99ebs-mounts /etc/rc5.d/S99ebs-mounts
{% endhighlight %}

Finally some cleanup. Let's removal all the packages that are wasting
resources!

{% highlight bash %}
apt-get -y remove sendmail apache2 bind9 samba nscd
{% endhighlight %}

Per Site Configuration
----------------------

We need a place to store all of our files. The mounted EBS volume is a
great place, as it won't be destroyed if/when the instance is killed,
and can be snapshot-ed separately from the rest of the system. Our
directory structure will look like /ebs/www/USERNAME/EXAMPLE.COM/. In
that folder, we will have a log folder and public web directory called
htdocs. In logs, we will have an access log and an error log. We will
first create a user to own our website directory, which will give some
added security between sites.

{% highlight bash %}
useradd -m -d /ebs/www/example -U example
mkdir -p /ebs/www/example/example.com/htdocs
mkdir /ebs/www/example/example.com/logs
touch /ebs/www/example/example.com/logs/access.log
touch /ebs/www/example/example.com/logs/error.log
chown -R example /ebs/www/example/
chgrp -R example /ebs/www/example
{% endhighlight %}

Now let's write our PHP pool for this site. Make sure the name in
brackets is unique, or you'll get errors. I also keep the general pool
around to dump less important sites into, so they share workers and
should use less resources overall.

{% highlight bash %}
nano /etc/php5/fpm/pools/example.pool
----

[example]
listen = 127.0.0.1:9001 # Increase this for each pool
user = example
group = example
pm = dynamic
pm.max_children = 5
pm.start_servers = 1
pm.min_spare_servers = 1
pm.max_spare_servers = 2
{% endhighlight %}

Now, we need to create a virtual host for our example site. This
provides us a way to host multiple different sites and domains from a
single server.

{% highlight bash %}
nano /etc/nginx/sites-available/example.com
--

server {
    listen 80;
    server_name example.com;
    access_log /ebs/www/example/example.com/logs/access.log;
    error_log /ebs/www/example/example.com/logs/error.log;
    root /ebs/www/example/example.com/htdocs;

    location / {
        index index.php index.html index.htm;
        location ~ \.php$ {
            fastcgi_pass 127.0.0.1:9001; # Increase this for each pool
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME /ebs/www/example/example.com/htdocs$fastcgi_script_name; # same path as above
            fastcgi_param PATH_INFO $fastcgi_script_name;
            include /etc/nginx/fastcgi_params;
        }
        # Static files
        if (-f $request_filename) {
            expires 30d;
            break;
        }
        if (!-e $request_filename) {
            rewrite ^(.+)$ /index.php?q=$1 last;
        }
    }

}
--
{% endhighlight %}

And to enable the site, we simply make a symbolic link between
sites-available and sites-enabled. With Apache, this is accomplished
with the a2ensite command, but nginx doesn't have such a command, so
we'll do it by hand (and create the command later..).

{% highlight bash %}
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com

#Now simply reload nginx to put the new site into effect
/etc/init.d/nginx reload
{% endhighlight %}

If you'd like a simple command do this, add the following lines to your
\~/.bash_aliases file. This will allow you to type "ngensite
example.com" and "ngdissite example.com" to enable and disable sites.
After you're satisfied with the sites enabled and disabled, just reload
nginx, as above.

{% highlight bash %}
ngensite () { ln -s /etc/nginx/sites-available/$1 /etc/nginx/sites-enabled/$1; }
ngdissite () { rm /etc/nginx/sites-enabled/$1; }

# For changes to effect, after saving the file simply type
bash
{% endhighlight %}

Finally, we need to download Wordpress! This is probably the easiest
part.

{% highlight bash %}
cd /ebs/www/example/example.com/htdocs/
wget http://wordpress.org/latest.tar.gz
tar xvf latest.tar.gz
mv wordpress/\* .
rm -rf wordpress
rm latest.tar.gz
{% endhighlight %}

Now just visit Example.com, and you will be able to finish the Wordpress
configuration. Use these settings (substitute your domain name):
Database Name: example User Name: example Password: dbpassword Database
Host: localhost Table Prefix: wp_

Finally, we need to secure the file with all these important passwords.

{% highlight bash %}
chmod 700 /ebs/www/example/example.com/htdocs/wp-config.php
{% endhighlight %}

Now just visit <http://example.com/wp-admin>, log in, and start your
blog! It'll be speedy and gorgeous. I think you'll instantly love nginx
and never look back at Apache.

Check back often for a tutorial on surviving the Slashdot/Digg effect
with some clever caching through Wordpress and nginx.

For each additional site, simply repeat everything below "Per Site
Configuration", reloading nginx and PHP-fpm each time.

With everything up and running like you want, you may want to stop the
server, and snapshot the instance's filesystem so you can spawn new
servers just like this one with a single click. I'll cover how to
snapshot our data EBS volume in the next article.
