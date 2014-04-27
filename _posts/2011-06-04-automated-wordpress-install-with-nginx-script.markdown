---
title: Automated Wordpress Install with Nginx Script


date: 2011-06-04 19:20

author: Josh

category: Articles

tags: nginx, Ubuntu, Wordpress

permalink:  automated-wordpress-install-with-nginx-script
---
It is a pain to follow my own tutorial over and over for each domain. So
I automated it! This script is tested ONLY on the official Ubuntu 10.04
32 bit AMI in a tiny instance, following the install instructions . I
have yet to test it anywhere else, but if it works for you, please leave
a comment at the bottom with your configuration. Also, if any Bash gurus
have tips for the script, your input would be very much appreciated.

This script creates a new user for the website and creates a home
directory on the mounted EBS volume to host the site. Then it creates a
new pool configuration, to give the site more secure execution. After
that, it creates a new virtual host in nginx, to tell the server how to
deal with the new site. It creates a MySQL user and database. Finally,
it downloads Wordpress, and installs it into the web directory, and
configures almost everything. When you visit the domain, you will be
asked for the site title, admin user (for the blog) and password, and
you email. After that, your blog is running!!

If you want to get the latest version of this script, you can [download
it
here](https://github.com/pcsforeducation/Sysadmin-scripts/tree/master/nginx-vhost).
I will post the current (very rough) script here, for posterity's sake.
It has basically no error checking, and probably isn't very portable. It
also exclusively works with nginx, PHP5-fpm, and MySQL, though this
should be easy to fix. This script also requires a few template files,
which are included at the bottom, and at the Github site.

**Note:**You will need the file /root/mysql with the root password on
the first line and nothing else. You should run "chmod 600 /root/mysql"
to ensure no one can read the root password.

{% highlight bash %}
#!/bin/bash
VHOST_DIR='/etc/nginx/sites-available'
USER_DIR='/ebs/www'
USERNAME=''
USERPASS=''
DOMAIN=''
MSYQLPASS=''
USER_TRUNC=''
DOMAIN_TRUNC=''
DB_PASS=''

RET=''
function sanity_check {
 if [ "$(id -u)" != "0" ]; then
 echo "Script must be run as root."
 exit 1
 fi
 if [[ $1 != 2 ]]; then
 echo $1
 echo "Usage: nginx-vhost.sh username example.com"
 exit 4
 fi

 egrep "^$USERNAME:" /etc/passwd >/dev/null
 if [ $? -eq 0 ]; then
 echo "$USERNAME exists!"
 exit 3
 fi

}

function password {
 RET=$(cat /dev/urandom | tr -cd [:alnum:] | head -c ${1:-16})
}

function setup_user {
 useradd -m -d $USER_DIR/$USERNAME -U $USERNAME

 password
 PASSWORD=$RET
 echo "$USERNAME:$PASSWORD" | chpasswd

 mkdir -p $USER_DIR/$USERNAME/$DOMAIN/htdocs
 mkdir $USER_DIR/$USERNAME/$DOMAIN/logs
 touch $USER_DIR/$USERNAME/$DOMAIN/logs/access.log
 touch $USER_DIR/$USERNAME/$DOMAIN/logs/error.log
 chown -R $USERNAME $USER_DIR/$USERNAME
 chgrp -R $USERNAME $USER_DIR/$USERNAME

 echo "Create user: $USERNAME with password: $PASSWORD"
}
function php_pool {
 if [ ! -f /etc/php5/fpm/port ]; then
 echo "Cannot access /etc/php5/fpm/port"
 exit 2
 fi

 # Grab the port from file, and increment.
 PORT=$(cat /etc/php5/fpm/port)
 echo $(($PORT + 1)) > /etc/php5/fpm/port

 cp /etc/php5/fpm/pool.template /etc/php5/fpm/pools/$DOMAIN
 sed -i "s|example.com|$DOMAIN|" /etc/php5/fpm/pools/$DOMAIN

 sed -i "s|PORT|$PORT|" /etc/php5/fpm/pools/$DOMAIN
 sed -i "s|user = example|user = $USERNAME|" /etc/php5/fpm/pools/$DOMAIN
 sed -i "s|group = example|group = $USERNAME|" /etc/php5/fpm/pools/$DOMAIN
 echo "Added $DOMAIN to the PHP pool"
}
function nginx_vhost {
 cp /etc/nginx/vhost.template /etc/nginx/sites-available/$DOMAIN
 sed -i "s|example.com|$DOMAIN|g" /etc/nginx/sites-available/$DOMAIN
 sed -i "s|username|$USERNAME|g" /etc/nginx/sites-available/$DOMAIN
 sed -i "s|PORT|$PORT|g" /etc/nginx/sites-available/$DOMAIN
 # Should probably sed through and replace /ebs/www with $USER_DIR

 # Enable the site
 ln -s /etc/nginx/sites-available/$DOMAIN /etc/nginx/sites-enabled/$DOMAIN
 echo "Enabled $DOMAIN in the web server"
}
function server_reload {
 /etc/init.d/php5-fpm reload
 /etc/init.d/nginx reload
 echo "Servers reloaded"
}
function prepare_mysql {
 # Generate dbuser password
 password
 DB_PASS=$RET

 # Truncate username (15 chars max) and dbname (63 chars max)
 USER_TRUNC=$(echo $USERNAME | cut -c1-15)

 # This should be a separate user with only create perms.
 echo "CREATE DATABASE $USER_TRUNC;
GRANT ALL PRIVILEGES ON $USER_TRUNC.* to '$USER_TRUNC'@'localhost' IDENTIFIED BY '$DB_PASS';" > $USERNAME.sql
 mysql -u root -p$(cat /root/mysql) < $USERNAME.sql
 rm $USERNAME.sql
 echo "Created MySQL user: $USER_TRUNC password: $DB_PASS database: $DOMAIN_TRUNC"
}
function install_wordpress {
 wget http://wordpress.org/latest.tar.gz
 tar xf latest.tar.gz
 mv wordpress/* $USER_DIR/$USERNAME/$DOMAIN/htdocs/
 cp $USER_DIR/$USERNAME/$DOMAIN/htdocs/wp-config-sample.php $USER_DIR/$USERNAME/$DOMAIN/htdocs/wp-config.php
 sed -i "s|database_name_here|$USER_TRUNC|g" $USER_DIR/$USERNAME/$DOMAIN/htdocs/wp-config.php
 sed -i "s|username_here|$USERNAME|g" $USER_DIR/$USERNAME/$DOMAIN/htdocs/wp-config.php
 sed -i "s|password_here|$DB_PASS|g" $USER_DIR/$USERNAME/$DOMAIN/htdocs/wp-config.php

 # Add in salts from the wordpress salt generator
 wget https://api.wordpress.org/secret-key/1.1/salt/
 sed -i "s/|/a/g" index.html
# cat index.html | while read line; do
 # sed -i "s|define('.*',.*'put your unique phrase here');|$line|" $USER_DIR/$USERNAME/$DOMAIN/htdocs/wp-config.php
# sed 1d $USER_DIR/$USERNAME/$DOMAIN/htdocs/wp-config.php
# done
 sed -i '/#@-/r index.html' $USER_DIR/$USERNAME/$DOMAIN/htdocs/wp-config.php
 sed -i "/#@+/,/#@-/d" $USER_DIR/$USERNAME/$DOMAIN/htdocs/wp-config.php
 rm index.html
 rm latest.tar.gz

 # Add in FTP stuff, even though that's not defined yet..

 # Own these new files
 chown -R $USERNAME $USER_DIR/$USERNAME/$DOMAIN/htdocs/*
 chgrp -R $USERNAME $USER_DIR/$USERNAME/$DOMAIN/htdocs/*
 # Make sure no one else can read this file
 chmod 700 $USER_DIR/$USERNAME/$DOMAIN/htdocs/wp-config.php

 #echo "Wordpress for $DOMAIN installed."
 #echo "Visit http://$DOMAIN/wp-admin/install.php to complete installation"
}

##############################################################################
# Start of program
USERNAME=$1
DOMAIN=$2

sanity_check $#
setup_user
php_pool
nginx_vhost
server_reload
prepare_mysql
install_wordpress
exit 0
{% endhighlight %}

Here is the PHP pool template. You can modify this to give all your
sites a different default.

{% highlight bash %}
nano /etc/php5/fpm/pool.template
--

[example.com]
listen = 127.0.0.1:PORT # Increase this for each pool
user = example
group = example
pm = dynamic
pm.max_children = 5
pm.start_servers = 1
pm.min_spare_servers = 1
pm.max_spare_servers = 2
{% endhighlight %}

You will also have to create a file that contains the next available
port number for FastCGI so nginx can communicate with PHP5-FPM.

{% highlight bash %}
nano /etc/php5/fpm/port
--
9001
{% endhighlight %}

Lastly, we need an nginx vhost template. This is where you can do a ton
of customization, according to the nginx documents. This template is
great for running Wordpress. I don't know how well it would run other
PHP projects, but it can certainly be customized to make them work.

{% highlight bash %}
nano /etc/nginx/vhost.template
--

server {
  listen 80;
  server_name example.com;
  access_log /ebs/www/username/example.com/logs/access.log;
  error_log /ebs/www/username/example.com/logs/error.log;
  root /ebs/www/username/example.com/htdocs;

  location / {
    index index.php index.html index.htm;
    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:PORT; # Increase this for each pool
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME    /ebs/www/username/example.com/htdocs$fastcgi_script_name;  # same path as above
        fastcgi_param PATH_INFO               $fastcgi_script_name;
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

That's it! Again, if this works for you or if you have a problem, please
leave a comment with your configuration! It will help me improve the
script for everyone. That's what open source is all about right?
