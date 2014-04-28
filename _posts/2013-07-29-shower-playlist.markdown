---
title: Shower Playlist


date: 2013-07-29 00:31

author: Josh

layout: default
category: Articles

tags: Android, Automation, Tasker

permalink: shower-playlist
---

I have a set of speakers in the bathroom that I use to play music and
podcasts while I shower. I had an extra NFC tag laying around, so I
decided to automate my music playing in the morning. With this, you walk
into the bathroom, plug the speakers into your headphone jack, put your
phone on the tag, and your morning playlist will start playing. Cool,
huh?

Here's what you need:

1.  [An NFC tag](http://amzn.to/14cY1xW)
2.  [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm) (costs
    \$2.99, but definitely worth it)
3.  [Auto
    Shortcut](https://play.google.com/store/apps/details?id=com.joaomgcd.autoshortcut&hl=en)
4.  [NFC Task
    Launcher](https://play.google.com/store/apps/details?id=com.jwsoft.nfcactionlauncher&hl=en)
5.  [Google Play Music](https://play.google.com/music/listen) (I'm using
    All Access) and a playlist
6.  Enable NFC on your phone. [Check here to see if your phone has
    NFC.](http://www.nfcworld.com/nfc-phones-list/) I'm using a Nexus 4.

Install all the apps above. In Tasker, go to Settings, Misc, and make
sure Allow External Access is checked. This will allow NFC Task Launcher
to launch tasker apps.

First, create a playlist in Google Play Music. I called mine...Morning.

In Tasker, go to the Tasks tab and hit the plus at the bottom. Let's
call this task "Shower". We're going to add a few steps that might not
be required, but work for me. Your mileage may vary. Add each of these
steps with the plus button at the bottom.

1.  App -\> Kill App -\> Play Music
2.  Plugin -\> AutoShortcut -\> Edit -\> Music Playlist -\> \*Your
    Playlist Name\*
3.  Media -\> Media Control -\> Next -\> Uncheck Simulate Media Button
4.  If your phone is rooted, continue on. If not, this is as far as you
    can go. You might have to hit the play button on your playlist to
    get the music going.
5.  Input -\> Dpad -\> Down
6.  Input -\> Dpad -\> Select -\> Repeat Times: 2

Now open up NFC Task Launcher.

1.  Slide over to My Tasks and hit the plus button.
2.  Select NFC as the trigger.
3.  Name it "Shower" again
4.  Hit the plus button to add an action
5.  Select Tasker, Tasker Task, then hit next
6.  Put in the name of your Tasker action ("Shower" in this example),
    then hit OK
7.  Hit the arrow in the top corner.
8.  Put your phone on your tag.
9.  Once it says your tag has been read correctly, hit the checkmark in
    top right corner.

That's it! Now you can just add songs to your playlist via your
phone/web browser and you'll get a playlist in the morning.

Ideally, I'd like a pair of bluetooth speakers that pair and start
playing. That way, all you do is walk in, put the phone on the tag, and
you're done.
