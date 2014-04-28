---
layout: default
title: Josh Gachnang's Blog
---

I am a software developer specializing in web apps and system administration (especially cloud systems). I primarily use Python/Django for my projects, and host them on Ubuntu servers or Google App Engine. I like to make things as simple as possible for the user. I love automating repetitive tasks and simplifying complex tasks to one click.

I would like to use my skills to improve the world. I am interested in automation, especially software that automates the mundane parts of our lives and general home automation.

I love coffee and beer. I've been brewing my own beer for nearly 2 years and recently started experimenting with roasting my own coffee.

If you're looking for my resume, [click here](/resume/).

### Recent Blog Posts:
{% for post in site.posts limit:10 %}
<li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}