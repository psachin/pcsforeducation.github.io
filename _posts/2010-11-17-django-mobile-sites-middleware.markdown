---
title: Mobile sites middleware

date: 2010-11-17 16:19

author: Josh

layout: default
category: Articles

tags: Django, Python

permalink:  django-mobile-sites-middleware
---
I love Django.  I also love CSS3.  Most of the sites that I'm developing
right now use CSS3, and therefore look terrible on many mobile devices
and Internet Explorer.  In this case, my options are either to make sure
every site gracefully falls back to IE-compatible HTML/CSS, or make a
separate site for the browser-impaired.  Well, call me malicious, but
I'm going to take the fast route and treat Internet Explorer users just
like mobile users and give them the less pretty (but still fully
functional) mobile site. I don't have the time make sure everything
looks good on standards compliant and non-compliant browsers, so
non-compliant desktop browsers (Internet Explorer, I'm looking at you)
and mobile phones are going to be treated the same way. Now the only
question is how to make Django do this easily.
<!--more-->
My implementation uses two parts: a middleware and a wrapper for the
Django shortcut render_to_response. I'm pretty sure this could be
compacted down into a single middleware file, but I wanted a wrapper for
render_to_response anyway, so that it passes in all my
context_processor and user variables in each view.

mobile_middleware.py

{% highlight python %}
MOBILE_USERAGENTS = ("2.0
MMP","240x320","400X240","AvantGo","BlackBerry",
"Blazer","Cellphone","Danger","DoCoMo","Elaine/3.0","EudoraWeb",
"Googlebot-Mobile","hiptop","IEMobile","KYOCERA/WX310K","LG/U990",
"MIDP-2.","MMEF20","MOT-V","NetFront","Newt","Nintendo Wii","Nitro",
"Nokia","Opera Mini","Palm","PlayStation Portable","portalmmm","Proxinet",
"ProxiNet","SHARP-TQ-GX10","SHG-i900","Small","SonyEricsson","Symbian OS",
"SymbianOS","TS21i-10","UP.Browser","UP.Link","webOS","Windows CE",
"WinWAP","YahooSeeker/M1A1-R2D2","iPhone","iPod","Android",
"BlackBerry9530","LG-TU915 Obigo","LGE VX","webOS","Nokia5800",)

user_agents_test = ("w3c ", "acs-", "alav", "alca", "amoi", "audi",
"avan", "benq", "bird", "blac", "blaz", "brew",
"cell", "cldc", "cmd-", "dang", "doco", "eric",
"hipt", "inno", "ipaq", "java", "jigs", "kddi",
"keji", "leno", "lg-c", "lg-d", "lg-g", "lge-",
"maui", "maxo", "midp", "mits", "mmef", "mobi",
"mot-", "moto", "mwbp", "nec-", "newt", "noki",
"xda", "palm", "pana", "pant", "phil", "play",
"port", "prox", "qwap", "sage", "sams", "sany",
"sch-", "sec-", "send", "seri", "sgh-", "shar",
"sie-", "siem", "smal", "smar", "sony", "sph-",
"symb", "t-mo", "teli", "tim-", "tosh", "tsm-",
"upg1", "upsi", "vk-v", "voda", "wap-", "wapa",
"wapi", "wapp", "wapr", "webc", "winw", "winw",
"xda-",)

import re

class MobileDetectionMiddleware(object):
    """
    Useful middleware to detect if the user is
    on a mobile device.
    """

    def process_view(self, request, view_func, view_args, view_kwargs):
        is_mobile = False;

        if request.META.has_key('HTTP_USER_AGENT'):
            user_agent = request.META['HTTP_USER_AGENT']

        # Test common mobile values.
        pattern =
            "(up.browser\|up.link\|mmp\|symbian\|smartphone\|midp\|wap\|phone\|windowsce\|pda\|mobile\|mini\|palm\|netfront)"
        prog = re.compile(pattern, re.IGNORECASE)
        match = prog.search(user_agent)

        if match:
            is_mobile = True;
            else:
            # Nokia like test for WAP browsers.
            # http://www.developershome.com/wap/xhtmlmp/xhtml_mp_tutorial.asp?page=mimeTypesFileExtension

        if request.META.has_key('HTTP_ACCEPT'):
            http_accept = request.META['HTTP_ACCEPT']

        pattern = "application/vnd\\.wap\\.xhtml\\+xml"
        prog = re.compile(pattern, re.IGNORECASE)

        match = prog.search(http_accept)

        if match:
            is_mobile = True

        if not is_mobile:
            # Now we test the user_agent from a big list.
            if user_agent in MOBILE_USERAGENTS:
                is_mobile = True

        test = user_agent[0:4].lower()
        if test in user_agents_test:
            is_mobile = True

        request.is_mobile = is_mobile
{% endhighlight %}

Create this file in the top directory of your project (the same folder
as urls.py). Then in add it to settings.py:

{% highlight python %}
MIDDLEWARE_CLASSES = (
'django.middleware.common.CommonMiddleware',
'django.contrib.sessions.middleware.SessionMiddleware',
'django.middleware.csrf.CsrfViewMiddleware',
'django.middleware.csrf.CsrfResponseMiddleware',
'django.contrib.auth.middleware.AuthenticationMiddleware',
'django.contrib.messages.middleware.MessageMiddleware',
'django.middleware.http.ConditionalGetMiddleware',
'django.middleware.gzip.GZipMiddleware',
'mobile_middleware.MobileDetectionMiddleware',
)
{% endhighlight %}

This will activate the middleware. Now anywhere in your views, you can
use request.is_mobile, which will be set to either True or False.

Now let's write the render_to_response wrapper. Put this at the top of
your views.py wherever you want to use it.

{% highlight python %}
def short_render(req, \*args, \*\*kwargs):
    kwargs['context_instance'] = RequestContext(req)
    template = args[0]
    template = 'mobile_' + template
    print template
    if req.is_mobile:
        return render_to_response(template, args[1])
    else:
        return render_to_response(\*args, \*\*kwargs)
{% endhighlight %}

Now anytime you want to render a template, simply do it like this:

{% highlight python %}
return short_render('template.html', {'form': form})
{% endhighlight %}

And it will render template.html for compliant browsers and
mobile_template.html for IE/mobile browsers.

If you want to make this work for only mobile browsers, simply remove
the statements marked off by \#IE Detection.

**Added 12/1/10**

While using this on my own site, I found it would be very nice for the
simple templates (such as the about page) to just choose which base
template file they extend.  First, I created a standard, desktop
base.html file, and then a mobile friendly (with jQuery Alpha)
mobile_base.html. If you have the above working, simple create a
mobile_template_processor.py in the root of your project:

{% highlight python %}
def MobileTemplate(request):
    if request.is_mobile:
        return {'extend_base': 'mobile_base.html'}
    else:
        return {'extend_base': 'base.html'}
{% endhighlight %}

Now let's install our new template processor in settings.py:

{% highlight python %}
TEMPLATE_CONTEXT_PROCESSORS = (
"django.contrib.auth.context_processors.auth",
"django.core.context_processors.request",
"django.core.context_processors.debug",
"django.core.context_processors.i18n",
"django.core.context_processors.media",
"django.contrib.messages.context_processors.messages",
"mobile_template_processor.MobileTemplate",
)
{% endhighlight %}

Now, to use this in a template, simply add this to the top:

{% highlight python %}
{% raw %}
{% extends extend_base %}
#Instead of this
{% extends "base.html" %}
{% endraw %}
{% endhighlight %}
