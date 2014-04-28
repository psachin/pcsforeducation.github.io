---
title: Django Performance Tuning

layout: default
category: Articles

tags: Django Python

permalink: django-performance-tuning
---
After one of my latest updates to SwoopSrch.com, all the sudden requests
were taking up to 11 seconds. Swoop works by making a set of all the
apartments that meet at least one criteria the user types in (usually
thousands), applying a weighting algorithm, sorting them, and returning
the top 100 values for display. Through some performance tuning, I was
able to get the requests down to about 1 second on average by applying
some simple techniques.

<!--more-->

Profile, Profile, Profile
-------------------------

Your app is slow. But why? Which functions? Is a function getting called
too many times? Is one function way too slow? You won't know until you
profile it. In your views.py, you can add a profile decorator that will
print out all your queries and a lot of data about functions (how many
times they are called, cumulative time taken, time per call, module/line
number, etc). Look for the highest cumulative time, ones being called
more than the length of your QuerySets, etc. Also check that you aren't
making too many queries to your database. I currently make about 7 after
optimization. I could get this down to about 3 with some clever
reworking, but none of the calls take very much time (less than 0.001
seconds), so I left them as they are. At one point, it was making over
1000. That is a problem.

To install and use Django Profiler and start profiling views:

{% highlight python %}
pip install django-profiler

# In views.py
from profiling import profile, Profiler

# On top of the view
@profile(stats=True)
{% endhighlight %}

If you just want to see your queries, Django gives you an easier way to
check them.

{% highlight python %}
from django.db import connection
from django.conf import settings

if settings.DEBUG == True:
    print connection.queries
{% endhighlight %}

You can also print the .query attribute of any QuerySet to see what
queries created it.

Use Querysets Effectively
-------------------------

You need to understand when Querysets are executed. For example,
intersecting QuerySets with the '|' operator does not execute a search,
but accessing the values does. If you then add more filters later,
you'll execute another search. Organizing your data flow is hugely
important. To get a better understanding, read When [QuerySets Are
Evaluated](https://docs.djangoproject.com/en/1.1/ref/models/querysets/#when-querysets-are-evaluated)
and [Database Access
Optimization](https://docs.djangoproject.com/en/1.1/topics/db/optimization/)
in the Django documentation. A big one is using ManyToMany fields
properly. If used improperly, you can get thousands of database calls,
and horrifying performance. Reconfiguring our database to not require
access to ManyToMany fields for normal queries shaved off about 5
seconds on its own. That was kind of shocking.

model_to_dict() or values()?
------------------------------

In my case, I needed all 100 of the Apartment objects to be a dictionary
before passing them back to the frontend as JSON. At first, I tried
using model_to_dict() on the complete object just before returning
them. At first, I was converting all of the objects to dicts with
model_to_dict(), but that was causing about a 0.3 second slow down.
Even after moving it, I was still getting a slow down of about 1/10 of
that, which isn't unbearable. Instead, I switched to calling
[.values()](https://docs.djangoproject.com/en/dev/ref/models/querysets/#django.db.models.query.QuerySet.values)
at the end of my query.

You need to be careful with this. If you have model functions, they will
be unavailable after you run .values(). For Swoop, Apartment objects
have a .address() function, combining the various address parts (house
number, street, etc) into a standard looking address for display. I
moved this function into a utility function, and experienced no more
slow downs. Good.

Defer When Possible
-------------------

Using
[defer()](https://docs.djangoproject.com/en/dev/ref/models/querysets/#django.db.models.query.QuerySet.defer)
is great if you have fields that are very large or ManyToMany fields,
you can defer them from getting retreived from the database by listing
them in defer() at the end of your query. A subtle note is that the
Django ORM tries to fill up its QuerySet cache, and part of that is
using
[select_related()](https://docs.djangoproject.com/en/dev/ref/models/querysets/#select-related).
select_related() prevents you from needing to hit the database again if
you follow a ForeignKey, but if you have too many objects, this can
massively slow down your queries. This accounted for about half of the
time.

As a user on reddit pointed out, you can also use
[only()](https://docs.djangoproject.com/en/dev/ref/models/querysets/#django.db.models.query.QuerySet.only)
if you only need a couple columns. Instead of excluding columns listed
in defer, it only grabs the ones you specify. In my case, there were
about 12 columns I need for every object, so defer made more sense. Your
mileage may vary.

Decimals Are Slow
-----------------

I originally stored the number of beds and baths as a DecimalField in
the database. When pulling them out of the database, they have to be
converted by the decimal module. According to [this StackOverflow
article](http://stackoverflow.com/questions/195116/python-decimal),
decimal is about 100 times slower than the float builtin. I changed the
fields in models.py to FloatFields, and issued an alter table to the
database, "alter table TABLE modify COLUMN double precision;". If you
need Decimals, you can instead try to defer out those fields when not
needed. A minor change, but it saved about 0.2 seconds. Nice.

Dates Are Slow
--------------

Running a datetime conversion when bringing objects out of a database is
slow when run a couple thousand times. Our database keeps track of when
apartments are available for rent (sometimes needed for searches), when
they were added to the database, and the last time they were updated (to
prioritize updates). The added and updated fields never need to be shown
to a user, so they are automatically added to the defer list. If the
available date isn't required, it is also deferred. This saved around
0.1 second again. Yay!

Trim The Queryset Before Doing Expensive Operations
---------------------------------------------------

Some of our semi-expensive operations, include some map preprocessing
and other backend preprocessing, were being applied before we trimmed
down the Queryset. Simply moving the trimming to 100 to much earlier
saved about 0.1 second. Awesome.

Slower Production vs Dev
------------------------

We are running on Amazon servers, with our MySQL database on a separate
server than our app servers. In development, we use sqlite3 for our
database, and it is local. Also, the Amazon servers use slower, using
2007 Opterons, where my laptop has a quad core Ivy Bridge. Obviously,
everything is going to act much faster on my laptop, and this initially
lulled me into believing the app would perform much better when
deployed. This is very hard to account for, but that is why you use
development and staging servers that mirror your production environment
as close as possible.

Caching
-------

One thing I have not implemented yet is a Memcache server. From what I
read, it is great for improving performance, but since Swoop uses a
custom weighting system for each search, I don't know that many queries
will be improved. Your app might be different, and Memcache might be the
first place to look for performance increases.

I hope these tips help you improve your app. This is not an exhaustive
list by any means. If you have more tips, add them in the comments and
I'll add them to the main article. Also, if you have any corrections,
please note them in the comments!
