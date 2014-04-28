---
title: Programming for Real Time - TriviaStats.com


date: 2013-04-23 16:41

author: Josh

layout: default
category: Articles

tags: Django, Logging, Python, TriviaStats

permalink: programming-for-real-time-triviastats-com
---

I recently deployed a website called
[TriviaStats.com](http://TriviaStats.com) for the world’s largest trivia
contested. Each year, [90FM in Stevens Point](http://90fmtrivia.org)
(the UW-Stevens Point radio station) hosts an over the radio trivia
contest. Teams listen to 8 questions an hour and call in answers for
points. There are also other events for points, such as a driving
scavenger hunt, a running scavenger hunt, and 1-2 second clips of songs
that must be identified. There were 376 teams this year and tens of
thousands of total players. Scores are posted on the website about 2
hours after the hour ends. So the scores for hour 2 get posted at the
end of hour 4. The website isn’t very easy to use, and you have to
constantly check to keep track of your scores. I thought I could make it
much easier. I wrote my site to scrape the score page and save the
scores for each hour. I then set up a form for people to register their
email/phone number and team name. When new scores were scraped, it would
notify them with their place and score. I also set up team pages that
let you see all your scores for this and past years at a glance, rather
than going through 50 pages to find the changes.

The main issue with writing a site like this is deal with scraping the
score page. It changes minorly every year, and the scraping must be
changed to meet it. Since this is a real time event, with players hungry
to find out their scores, it is a bit hard to fully test and be ready
for all the curveballs. Last year the site didn’t work, and I gave up
after a couple hours. It is very hard to play the trivia contest,
program, and drink all at the same time! Here are the things I think
made this year a success and things that could have been done better:

Logging!
--------

I would have had a much harder time diagnosing and fixing the site if I
wasn’t using a lot of logging and Sentry. Sentry is a server that you
can send all your logs to and easily sort and view the messages via a
web page. I was able to notice issues much faster and deal with them
much more easily because of Sentry. Also, seeing minor errors popping up
is reassuring. It is much more likely I made a small mistake somewhere
than I made no mistakes anywhere and there are no error logs being
generated. Even just shooting off an info logging message is reassuring.

Test like the event is happening
--------------------------------

Your tests should mirror the entire event, not just points in the event.
For example, a lot of my testing focused on getting past years data,
where all scores were available. The testing should have been in stages,
like “only hour 1 posted”, “hours 1 and 2 posted”, “hours 1 and 3
posted, 2 is missing”, etc.

Things are going to go wrong
----------------------------

In recent years, 90FM created a new page with something like
“resultsXX.html”, where XX was the hour number. This year, they only had
a results.html with all the scores that got changed each hour. That
means there will be no archival like past years, which allowed me to get
past years if there was a problem. I made an assumption that I could use
the XX in the page name to figure out which hour was posted. That was
easier than parsing “RESULTS FOR HOUR XX” at the top of the page. Once
the first scores got posted, I realized my mistake, and fixed it in a
few minutes. I’m very happy it was an easy fix, or I could have lost an
hour worth of scores!

You made a mistake somewhere
----------------------------

The assumption above lead me to an edge case that I had properly tested.
Because I had to change how the numbering worked, it messed up which
hours were getting emailed out as scores got posted. When hour 1 was up,
it emailed out hour 1. When hour 2 was up, it still emailed out hour 1.
Same for hour 3, which is when I realized and started fixing it. Once
you start getting curveballs thrown at you, all those little bugs in
your code are going to be much more apparent. Proper testing that really
keys in on edge cases will help avoid this, but you never know when
something is going to go horribly wrong.

Replay
------

If you hit an unexpected issue in your code, everything could break. For
example, if there was a problem while notifying players that I didn’t
catch, all subsequent notifications will be lost. I could go delete all
the scores for that hour, re-scrape them, and then hope notifications
don’t fail again. However, that isn’t elegant. Having checkpoints in
your code where you can replay an event as if it just happened makes
life much easier after you fix the issue. A smart way to do this would
be saving a small log object to the database, representing a successful
notification, or adding a field to each subscriber for “last hour
notified”. Another option is adding a command to re-send notifications
for a certain team/hour combo. I went for the even easier version:
resend all notifications. My thought was that it was more likely that
the whole thing would break rather than just sometimes breaking for
certain users. I created a Django management command for notifications
and a debugging flag to test the notifications before actually sending
them. In the end, it worked fairly well. A few people missed the first
few hours because they put in the wrong team name, so they got no
notification (which I manually corrected).

Another possible replay would have been downloading the page and running
the scraper against that. If there was an issue, I’d at least have the
page still around. If something had gone totally wrong and I didn’t fix
it before the next hour went up, I could have lost those scores forever.

Conclusion
----------

Overall, TriviaStats.com has 477 unique visitors (with 376 total teams)
and over 15k page view during the contest. A lot of people were very
interested and appreciative. Needless to say, I’m excited to improve it
for next year.

 
