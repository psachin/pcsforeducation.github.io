---
title: Building a Web Server on Amazon’s EC2

date: 2010-11-16 19:54

author: Josh

category: Articles

tags: EC2, Ubuntu

permalink:  building-a-web-server-on-amazons-ec2
---
Look at any Wiki, like Wikipedia. Pretty standard right? They all run
MediaWiki, a software installable in most shared hosting sites, like
my[Dreamhost](http://www.dreamhost.com/r.cgi?482511) account. However,
try writing articles for them.  I personally find it to be a cumbersome
process, both to set up, use, and write for.  Luckily, a few months ago,
I heard an [FLOSS Weekly](http://twit.tv/FLOSS) podcast about a fork
from MediaWiki, called Mindtouch.  Their product seemed really nice,
with awesome extensions and collaborative features, so I figured I’d
give them a try.  My first major roadblock was that you cannot just run
Mindtouch on any old shared hosting account.  You need at least a
Virtual Private Server (also available through Dreamhost for an extra
fee), or a full on Dedicated Server (much more expensive).  After
comparing all the possibilities, I decided the best bet was Amazon’s EC2
service.  I’ve used it a few times and it’s always been extremely
responsive, fairly easy to work with, and you only pay for what you use,
instead of a monthly cost.  They also have a multitude of tools
available to help you dynamically scale your infrastructure, so if one
day you get a ton of hits, you can just fire up a few more web servers,
and you’ll be golden.  I couldn’t find very much documentation on how to
do this (besides the normal web server set up guides), so here is my
tutorial!

**Amazon EC2**

[Amazon’s Elastic Compute Cloud](http://aws.amazon.com/ec2)(EC2) is a
service that lets you launch Amazon Machine Images (AMI’s), which are
modified Xen Virtual Machines.  In essence, it lets you start and stop a
virtual server running on their infrastructure, that acts almost
identical to any other server, but with a supremely fast Internet
connection (I’ve hit around 100Mbps to other servers for download
speed).  One big difference is the main root of the drive is reset every
time you terminate the instance (though it survives reboots).
 Therefore, we need to attach an [Elastic Block
Store](http://aws.amazon.com/ebs) (EBS) drive to make sure our data
survives outages and the like.  Then we are going to use symlinks to
have the data maintained on the EBS volume, but without changing tons of
configuration files.

If you don’t have an EC2 account, now would be a [good time to
start](http://awsmedia.s3.amazonaws.com/btn_signup_now.gif). Once you
do, sign into the [management
console](https://console.aws.amazon.com/ec2/), and go to these tabs on
the left hand side to prepare your web server.

**Security Groups**

First off, we are going to create a Security Group.  Security Groups are
ports that are opened through Amazon’s firewall and directed at your AMI
instance.  You can restrict the IP range, or leave it at 0.0.0.0/0 to be
publicly available.  The drop down menu on the left provides a lot of
pre-configured services.  I choose SSH (required if you want to
administrate the server) and HTTP (for serving over the web).  Ideally,
I should restrict SSH to my IP range, but I have a dynamic IP at home,
often control it from my phone (another dynamic IP) and work (2 set
ranges of IP’s).  Maybe in a future article!

**Key Pairs**

If you’ve only used password-based SSH, or never used SSH, this may be
an odd way to do it, but it is far more secure than password-based
logins.  When you create a new keypair, Amazon downloads a .pem file
(also called an identity file) to your computer.  It also creates
another file and stores it away in the web interface for making your own
AMI’s but we’ll cover that later.  Any time you want to connect to your
server, all you have to do is append “-i KEYPAIRNAME.pem” right after
“ssh” from the terminal and before the host name.  You can also insert
it into Putty if you’re a Windows user.

**Volumes (EBS)**

We need a place to store the files that need to survive shutting down
the machine.  Create a volume, but be generous.  Currently you can’t
grow a volume while it is running, so you will have to take down your
site momentarily to create more space, or start spreading data over
multiple volumes, which may undesirable. At \$0.10/GB a month, grabbing
an extra GB of space isn’t going to break the bank, but could definitely
save you some downtime.

**Launch an Instance**

For this example, I choose an Ubuntu Lucid Lynx 32 bit AMI.  Lucid Lynx
will be supported as a server OS for the next 5 years, which will help
me keep the reconfigurations to a minimum.  I also choose the official
image released by the Ubuntu folks, however, the AMI’s by Eric Hammond
at [Alestic.com](http://alestic.com/) have been exceedingly nice, too
(and often quicker to get bug fixes). The instance ID (when writing
this) is ami-2d4aa444 with
“ubuntu-images-us/ubuntu-lucid-10.04-i386-server-20100427″ as the
manifest. Once you hit launch, Amazon will process you request, and your
new server should be up and running in a minute or two.

**Configuration**

As soon as your instance’s status changes to “running” you can connect
to it with SSH/Putty.  Checkbox the instance you want to connect to, and
select “Connect” from the Instance Actions dropdown.  Copy the
information into the terminal, but be sure to change the username from
“root” to “ubuntu”.

The first thing to do is to get Ubuntu up to date (I would run “sudo su”
to become root first, otherwise append sudo to the commands)

{% highlight bash %}
apt-get -y update && apt-get -y upgrade
{% endhighlight %}

Then kick back and let it update from the Ubuntu mirrors running also on
EC2 (for best performance and no-cost downloading, since it is all
within EC2, or at the very worst minimal cost). First, let’s get our
directories in order format the disk.

{% highlight bash %}
mkdir /ebs
#Run through this, or look at how to automate it in the attached script
fdisk /dev/sdf  

For fdisk, use “l” to list the partitions, “d” to destroy a partition,
“n” to make a new partition, and “w” to write the configuration to the
disk. Unless you need a unique configuration, just accept all the
defaults to make one partition on the whole disk. Now we need to format
the drive. I prefer EXT4, though you may want XFS for various
applications.  Then we will mount it.

mkfs -t ext4 /dev/sdf1
mount -t ext4 $EBSPART /ebs>
mkfs -t ext4 /dev/sdf1  mount -t ext4 $EBSPART /ebs
{% endhighlight %}

In my script, I use the Linux command “sed” to edit all the users on the
system to prevent any from being able to login to a shell.  This should
provide some additional security. Next, we need to start installing all
the packages that will run our server.  We’re going to break them down
into groups, so they are easier to understand, and the script will be
easier to maintain. First we are going to make sure there will be
prompts throughout the process, then add the multiverse Ubuntu
repository and MindTouch repsoitory, so all of our packages are
accessible, then update our package lists.

{% highlight bash %}
export DEBIAN_FRONTEND=noninteractive
sed -ie ‘s_deb http://us-east-1.ec2.archive.ubuntu.com/ubuntu/ lucid main universe_deb http://us-east-1.ec2.archive.ubuntu.com/ubuntu/ lucid main universe multiverse_g’ /etc/apt/sources.list
echo ‘deb http://repo.mindtouch.com xUbuntu_10.04/’ >> /etc/apt/sources.list
apt-get update
{% endhighlight %}

Now let’s start the packages. First we will get required utilities for
other packages and MindTouch.

{% highlight bash %}
apt-get -y install fail2ban binutils cpp fetchmail flex gcc libarchive-zip-perl libc6-dev libcompress-zlib-perl libdb4.6-dev libpcre3 libpopt-dev lynx m4 make ncftp nmap openssl perl perl-modules unzip zip zlib1g-dev autoconf automake1.9 libtool bison autotools-dev g++ build-essential
{% endhighlight %}

Now we will get Apache and related tools.

{% highlight bash %}
apt-get -y install apache2 apache2-doc apache2-mpm-prefork apache2-utils apache2-suexec libexpat1 ssl-cert
{% endhighlight %}

At this point you may want to install DNS tools.  I am using Dreamhost
to point my domain name at the server and letting the server handle the
file part.  I won’t go in-depth on how to get DNS installed, but if you
want the packages, you can install:

{% highlight bash %}
apt-get -y install bind9 dnsutils
{% endhighlight %}

Now that we have Apache, we can install the “P” in LAMP server (PHP,
Python or Perl, depending on preference.  PHP in this example).

{% highlight bash %}
apt-get -y install libapache2-mod-php5 libapache2-mod-ruby libapache2-mod-python php5 php5-common php5-curl php5-dev php5-gd php5-idn php-pear php5-imagick php5-imap php5-mcrypt php5-memcache php5-mhash php5-ming php5-mysql php5-pspell php5-recode php5-snmp php5-sqlite php5-tidy php5-xmlrpc php5-xsl
{% endhighlight %}

We also need to get MySQL installed, however at the time of this
writing, doing a manual install came up with a lot of errors, and
rolling back to the previous version proved to have issues also, so I
let MindTouch install it as a dependency and configure it. Next up, we
need to keep the clock in sync with the world.

{% highlight bash %}
apt-get -y install ntp ntpdate
{% endhighlight %}

For security reasons, we should install tools to analyze our log files.

{% highlight bash %}
apt-get -y install webalizer vlogger
{% endhighlight %}

I removed AppArmor, only to reinstall it later.  Originally ISPConfig
was going to be a part of this server, but it required far too many
extra packages, so I’m just going to custom write something. You can
probably skip this part, and actually I would highly recommend that.

{% highlight bash %}
/etc/init.d/apparmor stop
update-rc.d -f apparmor remove
apt-get -y remove apparmor apparmor-utils
{% endhighlight %}

Now let’s get down to the MindTouch install.  First we need to install
PrinceXML (to convert pages to PDF documents).

{% highlight bash %}
cd /root
wget http://www.princexml.com/download/prince_6.0r8-1_i386.deb
dpkg -i /root/prince_6.0r8-1_i386.deb
rm /root/prince_6.0r8-1_i386.deb
{% endhighlight %}

Before the final MindTouch (called dekiwiki) install, we are going to
install all the recommend packages, and fix up any missing dependencies.

{% highlight bash %}
apt-get -y install imagemagick mono-runtime libmono-system-web2.0-cil curl php-pear php5-curl php5-gd html2ps html2text poppler-utils wv gs tidy links msttcorefonts cabextract aspell aspell-en
apt-get -y -f –force-yes install
apt-get -y install mono-devel
apt-get –force-yes -y install dekiwiki
{% endhighlight %}

We are going to install the Mozilla certificates next.  This will let
Mono (the underlying architecture in MindTouch) to trust the same set of
security certificates that a Mozilla browser does (like Firefox).

{% highlight bash %}
su mozroots
mozroots –import –sync
exit
{% endhighlight %}

There is a slight discrepancy between how MindTouch and Ubuntu treat
Ruby files, so we fix this error with this command:

{% highlight bash %}
sed -ie ‘s_application/x-ruby_#application/x-ruby_g’ /etc/mime.types
{% endhighlight %}

To get MindTouch to actually get run by Apache, we need to enable the
required Apache modules, and then enable the MindTouch site. After that,
we restart Apache for the changes to take effect.

{% highlight bash %}
a2enmod suexec rewrite ssl actions include proxy proxy_http
a2ensite dekiwiki
a2dissite default
/etc/init.d/apache2 restart
{% endhighlight %}

Now we are going to run a few commands to copy over the configuration
files to the EBS volume, and then make symbolic links back, so that we
don’t have to modify any configuration files.

{% highlight bash %}
##Move everything over to EBS, starting with web dir
mkdir /ebs/etc
if [ $INSTALL = "yes" ]; then
mv /var/www /ebs
else
rm -r /var/www
fi
ln –symbolic /ebs/www /var/www

##Move apache
/etc/init.d/apache2 stop
if [ $INSTALL = "yes" ]; then
mv /etc/apache2 /ebs
else
rm -r /etc/apache2
fi
ln –symbolic /ebs/apache2 /etc/apache2
/etc/init.d/apache2 start

#Move mime types definition
if [ $INSTALL = "yes" ]; then
mv /etc/mime.types /ebs/etc
else
rm -r /etc/mime.types
fi
ln –symbolic /ebs/etc/mime.types /etc/mime.types

#Move mysql DBs and restart
stop mysql
if [ $INSTALL = "yes" ]; then
mv /var/lib/mysql /ebs
else
rm -r /var/lib/mysql
fi
ln –symbolic /ebs/mysql /var/lib/mysql
start mysql

#Move deki and restart
/etc/init.d/dekiwiki stop
if [ $INSTALL = "yes" ]; then
mv /etc/dekiwiki /ebs
else
rm -r /etc/dekiwiki
fi
ln –symbolic /ebs/dekiwiki /etc/dekiwiki
{% endhighlight %}

And that should be it! Navigate to
[http://DNS\\_NAME/](http://DNS_NAME/) and you should see the
MindTouch configuration page! Follow the instructions on the MindTouch
setup page to get your install finished.  To find your DNS_NAME, open
up the AWS Management Console, and go to Instances. Click on the
instance you launched, and the bottom half of the screen displays data
about the instance.  Replace DNS_NAME with the Public DNS field. If you
want to have all this scripted, download one of the following scripts.
 Please read through and understand the scripts (you always should
understand what you’re going to run on your server). There’s also a lot
of commented out code, that you could enable to  Once you download it,
run the following commands to make the script executable and then
execute the script.

{% highlight bash %}
sudo chmod +x SCRIPT_NAME.sh
sudo sh ./SCRIPT_NAME.sh
{% endhighlight %}

The scripts will run on either an EC2 server or a normal server.  Make
sure to set the INSTALL option to the appropriate value (yes or no) for
EC2 servers.
