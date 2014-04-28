---
title: Nothing But Chromebook For A Week


date: 2011-07-01 10:36

author: Josh

layout: default
category: Articles

tags: Chromebook, ChromeOS

permalink: nothing-but-chromebook-for-a-week
---

As promised in my [Samsung Chromebook
review](http://www.servercobra.com/samsung-3g-chromebook-review/), I'm
using nothing but my Chromebook for a week. I'm a programmer, fledgling
system administrator, and blog writer. This isn't going to be easy. Not
only am I using this at home, but I'm leaving my workstations at work
shut off.

Day 1 - No Minecraft!?!?
------------------------

Day 1 was a normal day at work. I checked some tickets, check that
status of the computer labs, and stuff of that nature. Nothing really
intense, and nothing that required anything but the browser. I jammed
out on Pandora and Google Music. I checked a few docs in Google Docs.
Google+ worked great. I check email in GMail and our custom written
webmail client for campus/work (Aside: If some professors weren't so
worried about their work being stored in the cloud, I could do both from
GMail.). I didn't really notice I wasn't using a normal desktop, other
than not having dual monitors. I was too busy outside the office to get
any serious computing work done. That changed when I got home.

At home, I got a text from a friend who couldn't log into my Minecraft
server, because a new update for the clients, but I hadn't updated the
server. So, I dropped into Chrome OS's terminal (Ctrl + Alt + T) and
fired up crosh's (Chrome OS's extremely limited shell) ssh client. You
have to specify each parameter separately. For example:

{% highlight bash %}
crosh> ssh
ssh> host minecraft
ssh> user minecraft
ssh> connect

# And when you finish:
ssh> quit
crosh> exit
{% endhighlight %}

This is..annoying. I'm told if you flip the developer switch (a handy
requirement from Google for all Chromebooks), you get a better shell,
similar to bash, by typing "shell". I'll check that out later in the
week, as I'm just testing how this works for the normal user. Anyway, I
did what  I needed to do, and it was pretty easy once I was back to Bash
on my Ubuntu server. The only hard part I had was pasting the link to
the new server binary. To paste in the terminal, you have to middle
click. Middle click is awesome when you have a mouse, but is a 3 finger
press on the trackpad. For some reason, a 3 finger tap (like 1 and 2
finger taps, rather than clicks) doesn't work, and I'm forced to
actually click, which is relatively annoying.

From there, I proceeded to do a little administration work, and
everything was pretty good. You can have as many terminals as you want
open, which makes managing a couple computers quite simple. Once I got
some of my SSH keys downloaded, I could easily use those in SSH as well.
Just download the key into your downloads directory, and type "key
\<keyname\>" in the ssh prompt, no directory required. That was actually
easier than normal terminals.

One really nice thing I noticed was that Google Talk box hover over all
windows, not just in GMail like standard Chrome. If you click their
title bar, they minimize and stay out of the way. This is actually true
of all popups it seems, such as chatting with tech support. Nifty! I
usually keep a few windows open for business partners and my wife, to
make talking even easier. They minimize and you don't even notice
they're there, but pop up when you need them. The only thing I don't
like is the lack of keyboard shortcuts to jump between a chat box and
browser windows and back again. I'd expect Alt + Tab  to work, but no
such luck.

Battery life on Day 1 was superb. It was running for about 12 hours
total, with me using it around 8, and I still had 20% battery  left.
That is very impressive.

P.S. I did have to break out my Linux laptop to test that Minecraft was
working. Chrome OS isn't entirely perfect yet.

Day 2 - The Workstation
-----------------------

So I got to work early, and figured it would be a good time to play
around with my Chromebook and try a few things. The first thing I tried
was a new mouse. Holy crap, I love having a mouse with this thing. It's
not that I don't like the trackpad (it's one of the best I've used,
though I would like some freaking buttons. Is that too much to ask?),
but a mouse definitely makes it better, especially for extended use.
Then I grabbed a Microsoft ergonomic keyboard. Not too much of a change,
as I think the Chromebook has the nicest keyboard ever. If it had
backlit keys, I'd buy a couple just to rip out the keyboard for other
laptops. Then I grabbed the mini-VGA to VGA adapter, a 24 inch HP
monitor, and got it plugged in. Now I have a Chromebook workstation that
makes me sing. I feel like I can get real work done very efficiently.

[caption id="" align="aligncenter" width="400" caption="Awesome
Chromebook Workstation is Awesome"]|IMG_20110701_084529.jpg|[/caption]

My HTC Evo has been having some problems lately. I decided it was time
to back up everything, and wipe all of it. I plugged the phone into my
Chromebook with a USB cable, turned on USB storage, and opened the
Chrome OS file manager with Ctrl + M. Unfortunately, nothing came up.
Then I popped out the micro SD card, plugged it into an SD adapater, and
put it in the SD card slot of the Chromebook, but still, nothing showed
up. I had to break out another laptop to do it. I can't say for sure
that it was Chrome OS's fault, because my phone has been having a lot of
problems lately. It is a known issue, and should be fixed shortly. See
this [issue in Google
Code](http://code.google.com/p/chromium-os/issues/detail?id=14520).

Overall, I think this was a great day for my Chromebook. Plugging it
into a workstation really made this a more appealing laptop. When I need
it to be mobile, it is ultra portable. When I need to get more serious
work done, I can plug it into a workstation.

Day 3 - The Road Warrior
------------------------

My wife and I were driving up to see friends for the Fourth of July
today, so it was time to test out how the Chromebook worked on the road,
cut off from the rest of my computers. The 3G version comes with 100MB
of free data each month, and I have a Verizon 4G hotspot courtesy of
Google I/O, so I was set. It took a few minutes to set up 3G on the
Chromebook through Verzion (had to make an account, tried to upsell me
for more data per month), but after that, speeds were pretty good, and I
was able to get some basic work done. SSH was mostly usable for simple
commands, but writing a script from the command line was unusable. Oh
well. I switched over to my hotspot, so I could take advantage of
Madison's new 4G network. It was fast, and I couldn't tell I wasn't at
home. Frankly, I wind up using my 4G hotspot at places like libraries
and coffee shops, because it is faster. However, the response from SSH
was still slower than I'd like, so I wound up scripting in Google Docs
and copy/pasting into the terminal for testing. Eventually, as I got
towards northern Wisconsin, and my 3G signal started to get slower and
slower, and became too spotty for any real work. That's when I confirmed
a fear about the Chromebooks: Offline support isn't ready yet. According
to blog posts by Google Docs folks, it will be ready later this summer.

My thoughts on offline: Oh well. It really isn't a big deal for me. I
rarely work where there isn't Wifi, and when I do, I've got the 100MB of
data from Verizon in the Chromebook, a hotspot for a few more months,
and worse comes to worse, my phone is rooted and could be used as a
hotspot. Even on planes, I just pay the couple bucks for Wifi. If I were
a frequent flyer, I'd get the cheap monthly plan from AirTran. Overall,
not a big deal, but something I would like.

Due to the lack of 3G and offline support, I lost about an hour of work.
Instead, I actually talked to my wife and relaxed. Maybe Google is just
trying to save my marriage by not supporting offline.

I took 3 days off because I was on vacation and didn't do any work on a
computer (I did play the tutorial for a game and show off Minecraft).
Day 4 resumes on July 6, rather than the 3rd.

Day 4 - Back to Work
--------------------

After a fun Fourth, I finally got back to work. Well, it was mostly
work. First off, I wanted to start migrating from Facebook over to
Google+, starting with pictures. I stumbled on some great software
called Move2Picasa.org, but the site said they were Tech Crunch'd, so it
would be very slow. My next plan was to download all the files from
Facebook (Account Settings -\> Download Your Information). Then I should
be able to upload my pictures to Picasa with the file manager. However,
it all comes in a zip file. Yet another minor tool missing from Chrome
OS, which should have been easy to implement. Then I went search around
online for an easy unzip tool. I found WobZIP.com, but after unzipping
my file, I would have had to download each file individually. At a
couple hundred pictures, I'll just wait a few days and do it on my
desktop.

**Edit:**You can now upload .ZIP and .RAR files to Google Docs and they
will be unzipped. You could then download the entire folder, and upload
it to Picasa. A bit more work than a standard OS, but I'm hoping tighter
integration of Google Docs with Picasa will come soon.

Today I figured it would be a great day to try out some programming. I
mainly develop in Python and Bash. Bash programming was fairly
easy...when I SSH'ed into a server and wrote it there. Similarly, I
could do the same thing in Python, but frankly, I'd rather not. There is
a great app called [Cloud9 IDE](http://cloud9ide.com). I met 2 of the
developers at Google I/O, and their demo was impressive. However, the
software wasn't fully rolled out yet, so I couldn't get much done. I was
hoping for deployment to App Engine, where I host most of my web apps,
but no luck. On the high side, they say it'll be rolling out soon, so
that should make development great. The interface is great, has full git
support, and pretty much does everything I'd want a Python IDE to do,
minus the deployment issue. Almost there Chrome OS!

When I got home from work, I tested out video calling people with Google
Talk. It worked great, with no plugins to install or anything! The
machine got a bit slow with a ton of tabs open, but that's to be
expected. I also tested Hangout in Google+, and it worked like a charm.
Again, some slow down, but nothing major. This would be great for family
members, because getting them to use Skype takes some work, and then we
could more easily keep in touch.

Well, the Chromebook worked pretty well today for some script
development in Bash, thanks to SSH. However, it fell on it's face for
Python development. Javascript development would work on Cloud9 I
believe, as well as HTML/CSS, but I think that is the limit. I suppose I
could SSH to the school's servers and write my C code if I needed, since
IDE's don't seem to add much to it, and frankly I always write C in
nano. I spent more of the day working on blogs and social networking.
Obviously, the Chromebook wasn't really meant for developers (yet!!),
but for most people, it'd be pretty good.

### Day 5 - Another Day in the Life of a Chromebook

Day 5 has me at work again.  I've been searching for cloud tools to get
work done, and testing them out. Here's my findings!

**Music: **I've been jamming out to Pandora and Google Music most of the
day. A huge, huge problem for Chrome OS here, is how do I buy music!!
Many people are used to desktop apps for this kind of stuff. First, you
can buy your music at the [Amazon MP3
Store](http://www.amazon.com/gp/redirect.html?ie=UTF8&location=http%3A%2F%2Fwww.amazon.com%2FMP3-Music-Download%2Fb%3Fie%3DUTF8%26node%3D163856011%26ref_%3Dsa_menu_mp3_str1%23&tag=serve01-20&linkCode=ur2&camp=1789&creative=390957),
and save it to their Cloud Drive. Then you'll be able to stream it.
Frankly, I think this is the way most people should buy music,
Chromebook or not. For all of your existing music, you can either upload
it to Cloud Drive, or you can use Google Music. I'm a big fan of Google
Music, so I generally buy music from Amazon, then upload it to Google
Music. Both allow you to stream them online and have Android apps, so it
really comes down to preference.

**Picture Editing:** The app [Picnik](http://picnik.com) is a very
impressive photo editor entire in the web. It allows you to do basic
editing tasks, such as crop, rotate, resize, exposure, colors, etc.
There is also a handy AutoFix tool. The best part is you can import
pictures from photo sites, such as Facebook, Flickr, and Picasa to edit
them, and then export them again. No more downloading! If you go
premium, you can use multiple sites at the same time.

**Video:**I was very disappointed to see that Netflix isn't supported at
launch. However, Hulu is, so my TV shows are covered. Also, renting
movies from YouTube is awesome, and works great on the Chromebook.
Quality is great and snappy.

**Documents:**The easy and obvious way to write documents in Chrome OS
is Google Docs. You can edit documents with multiple people, easily
share, and get work done. If you're used to Microsoft Office, you can
use [Microsoft's Live Office](http://www.officelive.com/), which is
basically Microsoft Office, online, for free. Both allow you to save all
your docs to the cloud, so if you need to use another computer, they are
all easily accessible.

Day 5 didn't have as much notable work to talk about, but looking at all
the apps, I think most people can get most of what they do on a daily
basis done on one of these gorgeous laptops. I asked all my
Facebook/Twitter followers for a list of the applications they use most
often at work and at home, so hopefully I'll have a list of what you can
and cannot do on Chrome OS soon. The main things you can't do so far are
use the Adobe Suite, do most art-related things, download torrents, or
do some system administration tasks.

### Day 6 - Install Ubuntu!

On Day 6, I decided it was time to push the Chromebook. I've read
articles about installing Ubuntu on the CR-48 (the beta version of the
Chromebook), and it sounded pretty awesome, so I'm going to give it a
try. It is kind of breaking with the idea of "Nothing But Chromebook For
a Week", but I think this simple mod will expand the potential user base
for Chromebooks, so let's do it anyway. Check t[he article I
followed](http://chromeos-cr48.blogspot.com/2011/04/ubuntu-1104-for-cr-48-is-ready.html)
if you have questions.

The first thing you need to do is enable developer mode. Luckily, Google
has demanded that every Chromebook have a Developer Mode switch, so pop
open the tab on the right side of the Samsung Chromebook, and flip the
little switch. I'm not sure where it is on the Acer model, and it
appears to be under the battery on the CR-48. Reboot your laptop, and
it'll wipe your stateful partition. Let me say that again: IT WILL WIPE
YOUR DOWNLOADS FOLDER! So back it up if you have anything useful there.
Then go through the normal set up process, and then hit Ctrl + Alt + -\>
(the forward arrow where F2 should be). This will drop you into the
developer shell. Log in as "chronos" for your user. Here you'll have
full access to a real SSH client, and most other bash tools (finally!).
Now, we need to enable the developer BIOS, so it'll let us boot
non-Ubuntu distros. Simply type "**chromeos-firmwareupdate
--mode=todev**", and you'll be good. Reboot once more for good measure,
and let's jump into install our new OS.

Jump back into the developer shell and log in as chronos again. To
download and run the script to get everything install, run "**wget
<http://goo.gl/hnkxo>; sudo sh hnkxo"**. First, it is going to prompt
you for how much space you want to allocate to Ubuntu. The stateful
partition is under 11GB, so for best results, I'd recommend choosing
9GB. That'll leave over 1GB for caching and downloads in Chrome OS. It
is going to take quite some time, because it has to download 5GB of
compressed data, so hopefully you're on some speedy wifi. Once it
finishes, it should reboot for you. If not, reboot, and you'll be
dropped into Ubuntu! The username is "user" with password "user" (super
security!). Create a new user, modify everything to how you want it (Go
into Mouse in the Preferences menu to enable 2 finger scroll and tap to
click). To make this stick, we need to modify which kernel is more
important.

If you run "sudo cgpt show /dev/sda" in the terminal (Ctrl + Alt + T),
you'll get a list of the partition table. KERN-A and KERN-B are the 2
Chrome OS kernels (so Chrome OS can update the kernel not in use, and
switch on next reboot. If the update fails, you can just fall back to
the previous kernel). KERN-C is a copy of the Chrome OS kernel and will
be used for Ubuntu. We need to set KERN-C's priority to a number higher
than the A and B. That way, when we are in developer mode, we'll boot
Ubuntu, and when we are in normal mode, it'll boot Chrome OS. Best of
both worlds, right? To do this, simply type "**sudo cgpt add -i 6 -P 5
-S 1 /dev/sda**".

Finally, to resize your partition to fill the whole 9GB, you'll just
need to run "**sudo resize2fs -p /dev/sda7**". Because the filesystem
for Ubuntu, ext4, is awesome, it'll do this on the fly while you're
still using Ubuntu. Cool huh?

That should cover everything on how to get Ubuntu installed. From there,
I was able to do just about everything I needed, especially development
related tasks. The hardware in the Chromebook is definitely fast enough
to do everything I need for development. I still switch back to Chrome
OS for most of what I do day to day, because it does most of what I
want. Chrome OS in developer mode gives me access to bash, and from
there I can do even more.

This is one area where buying a different netbook and installing
ChromeOS on it might be a better idea. You'd be able to install any
version of Ubuntu that you want, or even Windows if you so desire. I was
doing this for a while before the Chromebooks were released, and I'm
hoping I can get ChromeOS running on some older desktops for family
members (no more support calls? Yes please!). Another option besides
buying a Chromebook is to pick up a Mini 9 or 10v netbook through the
Dell [Laptop deals](http://www.dell.com/us/p/popular-laptop-deals) page
for cheap, and then install ChromeOS directly onto it. I've always like
the Dell Minis, and their battery life is absolutely superb.

Day 7 - The Conference

For Day 7, I went to Barcamp Chicago, an unconference where the
attendees are the speakers and everything is very free form. On the way
there, I decided to update a few articles, write a few new ones, and do
a little maintenance on my blog. My friend was driving, so I busted out
my Chromebook, and connected to the 100MB of free data. I disabled a few
Chrome extensions that might use a lot of data (Twitter and such), but
other than that, I used my Chromebook like any other laptop on standard
Wifi. Almost all of what I did was exactly the same as normal. I still
had some issue getting work done via SSH over 3G, but it was manageable.
This was probably because I was headed towards Chicago, rather than
towards northern Wisconsin. My battery life was fantastic, even when
using 3G. I was able to write for the 2 hour drive, then take notes for
a few hours at the conference, and still have plenty of battery life
left over. During the whole day, which included a bit of exploring
Chicago, I had my Chromebook with me. It was quite light, especially
without the charger which is rarely necessary. This is in stark contrast
to my Dell 13" laptop, which is so heavy by the time I make the 20
minute walk to class, my back starts to hurt.

Being a college student, I'm very used to taking notes any time someone
gets in front of a group of people and starts talking. There were
various talks, many of which had valuable points which I wrote down for
future reference. Taking notes was just as simple as when I took notes
on my other notebooks in classes. I believe this will be the perfect
laptop for students. Many schools use Google Apps already, so this is a
great compliment. I was easily able to check my school email, and my
GMail quite easily. Basically, during the whole day, Chrome OS was all I
needed, except to show off that I had Ubuntu installed on it.

### Conclusion

The Chromebook can do most of the things most users will need. For
people that use less specialized applications, such as intense gaming,
Java-based apps, and system administration apps, the Chromebook could be
a primary computer. However, it is designed to be a secondary computer,
and in that role, it does everything necessary. With the 3G version or a
hot spot, it is the ultimate in portable computing. With the coming
offline capabilities, it will be even more useful on the go. The
hardware itself is very well designed, and pleasing to use, often more
so than other, more fully-functional laptops. I believe this is the only
kind of computer I will encourage many of my family members and some
friends to buy, because it is exceedingly simple, has zero management,
and web apps simplify much of my life. The Chromebook automatically
updates itself without notification, it checks itself on boot, which
should mitigate most viruses, and the lack of installed apps means no
required maintenance. With the addition of "Chromoting", along with apps
by VMWare and Citrix, Chromebooks will soon be able to connect to any
apps that aren't available online. Chrome OS is impressive, and will
only become more impressive as more apps become available online.

### More!

Don't just take my word on it, check out reviews on Amazon. Also, the
cheaper Acer Chromebook is at the bottom.
