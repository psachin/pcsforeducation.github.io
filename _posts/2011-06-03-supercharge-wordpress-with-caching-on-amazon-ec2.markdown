---
title: Supercharge Wordpress with Caching on Amazon EC2


date: 2011-06-03 15:26

author: Josh

layout: default
category: Articles

tags: EC2, Ubuntu, Webserver, Wordpress

permalink: supercharge-wordpress-with-caching-on-amazon-ec2
---

In my previous article, I showed you how using nginx and PHP5-fpm could
cut down on your memory usage and speed up your site. However, it wasn't
fast enough. We are going to make it even faster. To do this, we are
going to use multiple types of caching and a CDN (Amazon S3) to make our
data closer to our users, among a few other tweaks. This will also
shrink all of our code (HTML, CSS, and Javascript), much like the
PageSpeed module Google provides for Apache.

If these tips and tweaks help your site, post your results in the
comments! And as always, feel free to ask questions or suggest
improvements.

First, we will install APC to cache PHP opcodes. This means less
processing if the object is found in the cache, at the expense of a
little extra memory usage (default 32MB). Since tiny instances have such
limited CPU usage, this is a good trade of.

**Note:** All commands are run as root. If you are not root, simply type
"sudo su" at the command prompt.

{% highlight bash %}
apt-get install php-apc
{% endhighlight %}

And basically...That's it! The defaults are very good.

In the last article we covered how to enable the MySQL query cache. If
you didn't follow that tutorial, here's how to set it:

{% highlight bash %}
nano /etc/mysql/my.conf
----
# Ensure these lines are uncommented.
query_cache_limit = 1M
query_cache_size = 16M
{% endhighlight %}

Now login to your Wordpress and install the W3-Total-Cache plugin. Then,
in the plugins page, hit "Settings" under W3-Total-Cache. We are going
to have to add a few rules to nginx.conf to make it work as well as it
can. It is originally written for Apache, but the following changes will
make it work even better. There are a ton of changes to be made!
Everything between Begin Ch

{% highlight bash %}
# First we will install PHP Curl extensions so we can use Amazon S3 as our CDN
apt-get -y -q install php5-curl
/etc/init.d/php5-fpm restart

# And then edit our domain configuration
nano /etc/nginx/sites-available/example.com
----
server {
  listen 80;
  server_name joshgachnang.com;
  access_log /ebs/www/joshgachnang/joshgachnang.com/logs/access.log;
  error_log /ebs/www/joshgachnang/joshgachnang.com/logs/error.log;
  root /ebs/www/joshgachnang/joshgachnang.com/htdocs;

  location / {
    index index.php index.html index.htm;

################################################
# Begin Changes
################################################
# BEGIN W3TC Page Cache core
rewrite ^(.*\/)?w3tc_rewrite_test$ $1?w3tc_rewrite_test=1 last;
set $w3tc_rewrite 1;
if ($request_method = POST) {
    set $w3tc_rewrite 0;
}
if ($query_string != "") {
    set $w3tc_rewrite 0;
}
set $w3tc_rewrite2 1;
if ($request_uri !~ \/$) {
    set $w3tc_rewrite2 0;
}
if ($request_uri ~* "(sitemap\.xml(\.gz)?)") {
    set $w3tc_rewrite2 1;
}
if ($w3tc_rewrite2 != 1) {
    set $w3tc_rewrite 0;
}
set $w3tc_rewrite3 1;
if ($request_uri ~* "(\/wp-admin\/|\/xmlrpc.php|\/wp-(app|cron|login|register|mail)\.php|wp-.*\.php|index\.php)") {
    set $w3tc_rewrite3 0;
}
if ($request_uri ~* "(wp\-comments\-popup\.php|wp\-links\-opml\.php|wp\-locations\.php)") {
    set $w3tc_rewrite3 1;
}
if ($w3tc_rewrite3 != 1) {
    set $w3tc_rewrite 0;
}
if ($http_cookie ~* "(comment_author|wp\-postpass|wordpress_\[a\-f0\-9\]\+|wordpress_logged_in)") {
    set $w3tc_rewrite 0;
}
set $w3tc_ua "";
if ($http_user_agent ~* "(2\.0\ mmp|240x320|alcatel|amoi|asus|au\-mic|audiovox|avantgo|benq|bird|blackberry|blazer|cdm|cellphone|danger|ddipocket|docomo|dopod|elaine/3\.0|ericsson|eudoraweb|fly|haier|hiptop|hp\.ipaq|htc|huawei|i\-mobile|iemobile|j\-phone|kddi|konka|kwc|kyocera/wx310k|lenovo|lg|lg/u990|lge\ vx|midp|midp\-2\.0|mmef20|mmp|mobilephone|mot\-v|motorola|netfront|newgen|newt|nintendo\ ds|nintendo\ wii|nitro|nokia|novarra|o2|openweb|opera\ mobi|opera\.mobi|palm|panasonic|pantech|pdxgw|pg|philips|phone|playstation\ portable|portalmmm|ppc|proxinet|psp|qtek|sagem|samsung|sanyo|sch|sec|sendo|sgh|sharp|sharp\-tq\-gx10|small|smartphone|softbank|sonyericsson|sph|symbian|symbian\ os|symbianos|toshiba|treo|ts21i\-10|up\.browser|up\.link|uts|vertu|vodafone|wap|willcome|windows\ ce|windows\.ce|winwap|xda|zte)") {
    set $w3tc_ua _low;
}
if ($http_user_agent ~* "(acer\ s100|android|archos5|blackberry9500|blackberry9530|blackberry9550|blackberry\ 9800|cupcake|docomo\ ht\-03a|dream|htc\ hero|htc\ magic|htc_dream|htc_magic|incognito|ipad|iphone|ipod|kindle|lg\-gw620|liquid\ build|maemo|mot\-mb200|mot\-mb300|nexus\ one|opera\ mini|samsung\-s8000|series60.*webkit|series60/5\.0|sonyericssone10|sonyericssonu20|sonyericssonx10|t\-mobile\ mytouch\ 3g|t\-mobile\ opal|tattoo|webmate|webos)") {
    set $w3tc_ua _high;
}
set $w3tc_ref "";
if ($http_cookie ~* "w3tc_referrer=.*(google\.com|yahoo\.com|bing\.com|ask\.com|msn\.com)") {
    set $w3tc_ref _search_engines;
}
set $w3tc_ssl "";
if ($scheme = https) {
    set $w3tc_ssl _ssl;
}
set $w3tc_enc "";
if ($http_accept_encoding ~ gzip) {
    set $w3tc_enc .gzip;
}
if (!-f "$document_root/ebs/www/joshgachnang/servercobra.com/htdocs/wp-content/w3tc/pgcache/$request_uri/_index$w3tc_ua$w3tc_ref$w3tc_ssl.html$w3tc_enc") {
    set $w3tc_rewrite 0;
}
if ($w3tc_rewrite = 1) {
    rewrite .* "/ebs/www/joshgachnang/servercobra.com/htdocs/wp-content/w3tc/pgcache/$request_uri/_index$w3tc_ua$w3tc_ref$w3tc_ssl.html$w3tc_enc" last;
}
# END W3TC Page Cache core

# BEGIN W3TC Page Cache cache
location ~ /ebs/www/joshgachnang/servercobra.com/htdocs/wp-content/w3tc/pgcache.*html$ {
    add_header X-Powered-By "W3 Total Cache/0.9.2.2";
    add_header Vary "Accept-Encoding, Cookie";
}
location ~ /ebs/www/joshgachnang/servercobra.com/htdocs/wp-content/w3tc/pgcache.*gzip$ {
    gzip off;
    types {}
    default_type text/html;
    add_header X-Powered-By "W3 Total Cache/0.9.2.2";
    add_header Vary "Accept-Encoding, Cookie";
    add_header Content-Encoding gzip;
}
# END W3TC Page Cache cache

# BEGIN W3TC Browser Cache
gzip on;
gzip_types text/css application/x-javascript text/richtext image/svg+xml text/plain text/xsd text/xsl text/xml image/x-icon;
location ~ \.(css|js)$ {
    add_header X-Powered-By "W3 Total Cache/0.9.2.2";
}
location ~ \.(html|htm|rtf|rtx|svg|svgz|txt|xsd|xsl|xml)$ {
    add_header X-Powered-By "W3 Total Cache/0.9.2.2";
}
location ~ \.(asf|asx|wax|wmv|wmx|avi|bmp|class|divx|doc|docx|exe|gif|gz|gzip|ico|jpg|jpeg|jpe|mdb|mid|midi|mov|qt|mp3|m4a|mp4|m4v|mpeg|mpg|mpe|mpp|odb|odc|odf|odg|odp|ods|odt|ogg|pdf|png|pot|pps|ppt|pptx|ra|ram|swf|tar|tif|tiff|wav|wma|wri|xla|xls|xlsx|xlt|xlw|zip)$ {
    add_header X-Powered-By "W3 Total Cache/0.9.2.2";
}
# END W3TC Browser Cache

# BEGIN W3TC Minify core
rewrite ^/ebs/www/joshgachnang/servercobra.com/htdocs/wp-content/w3tc/min/w3tc_rewrite_test$ /ebs/www/joshgachnang/servercobra.com/htdocs/wp-content/w3tc/min/index.php?w3tc_rewrite_test=1 last;
set $w3tc_enc "";
if ($http_accept_encoding ~ gzip) {
    set $w3tc_enc .gzip;
}
if (-f $request_filename$w3tc_enc) {
    rewrite (.*) $1$w3tc_enc break;
}
rewrite ^/ebs/www/joshgachnang/servercobra.com/htdocs/wp-content/w3tc/min/(.+\.(css|js))$ /ebs/www/joshgachnang/servercobra.com/htdocs/wp-content/w3tc/min/index.php?file=$1 last;
# END W3TC Minify core

# BEGIN W3TC Minify cache
location ~ /ebs/www/joshgachnang/servercobra.com/htdocs/wp-content/w3tc/min.*\.js$ {
    types {}
    default_type application/x-javascript;
    add_header X-Powered-By "W3 Total Cache/0.9.2.2";
    add_header Vary "Accept-Encoding";
}
location ~ /ebs/www/joshgachnang/servercobra.com/htdocs/wp-content/w3tc/min.*\.css$ {
    types {}
    default_type text/css;
    add_header X-Powered-By "W3 Total Cache/0.9.2.2";
    add_header Vary "Accept-Encoding";
}
location ~ /ebs/www/joshgachnang/servercobra.com/htdocs/wp-content/w3tc/min.*js\.gzip$ {
    gzip off;
    types {}
    default_type application/x-javascript;
    add_header X-Powered-By "W3 Total Cache/0.9.2.2";
    add_header Vary "Accept-Encoding";
    add_header Content-Encoding gzip;
}
location ~ /ebs/www/joshgachnang/servercobra.com/htdocs/wp-content/w3tc/min.*css\.gzip$ {
    gzip off;
    types {}
    default_type text/css;
    add_header X-Powered-By "W3 Total Cache/0.9.2.2";
    add_header Vary "Accept-Encoding";
    add_header Content-Encoding gzip;
}
# END W3TC Minify cache

################################################
# End changes
################################################
    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9001;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME    /ebs/www/joshgachnang/joshgachnang.com/htdocs$fastcgi_script_name;  # same path as above
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
{% endhighlight %}

Now login to your Wordpress admin panel. Go install the W3 Total Cache
plugin. Then Click the new Performance Panel on the left. Here is the
list of caches I enabled and the page cache method:

-   Page Cache - Opcode: APC
-   Minify - Opcode: APC
-   Database Cache - Opcode: APC
-   Content Delivery Network - Amazon Simple Storage Service
-   Browser Cache

Then hit deploy at the top, and test it to make sure its working. After
that, click "Content Delivery Network" near the top. ﻿Fill in the Access
key ID and Secret key. Save your settings, then fill in the Bucket: with
your domain name (example.com), and hit Create Bucket.  Hit Test S3
upload just in case.

Now scroll all the way to the top. We need to upload our whole media
library. For some reason, when I hit "export the media library", it
didn't want to upload. Instead hit each of "wp-includes", "theme files",
and "custom files", and hit "Start" on each. This should upload all your
files up to S3. Reload your site, and see how fast it is! Dig through
all the settings, understand them, and tweak them, with a lot of
testing.

A great way to test the speed on your server is the "ab" or Apache
Benchmark. Here's a quick tutorial.

{% highlight bash %}
# To install Apache Benchmark:
apt-get -y -q install apache2-utils

# To run ab, I find this is a good test. Add a few magnitudes of 10 to the number of runs to really add some extended stress
# This will run 10 requests concurrently, each hitting the site 100 times. Bump up 100 to 1,000 or 10,000 for extended tests.
ab -kc 10 -n 100 http://example.com/
{% endhighlight %}
