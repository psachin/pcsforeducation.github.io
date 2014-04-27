---
title: The Only Guide to Git You'll Ever Need


date: 2011-06-12 23:48

author: Josh

category: Articles

tags: Git

permalink: the-only-guide-to-git-youll-ever-need
---

If you've ever used Git, or Subversion, or any version control system
really, you know there is usually a decent learning curve.  I stumbled
across[this guide](http://bit.ly/iUAI80) on reddit, and it sums up
everything I've been trying to teach myself for the past few months
quite concisely. I would recommend reading through everything in that
article. Maybe even print out some of them to post on your office wall
when you start with Git. A couple thing I would add though.

.gitignore
----------

Git ignore is a file which contains patterns of files you want to
ignore. When adding files, if git finds that any meet these patterns,
they won't be added. This is a great idea for text editor saves,
compiled files, etc. Here's a couple good things to add in my
experience:

{% highlight bash %}
nano /path/to/project/.gitignore
----
# Text editor files (check your favorite editor to see what it does
\*~
# Compiled Python files
\*.pyc
# Compiled C files
\*.o
# Sometimes output files of compiled C
\*.out
# Leftovers from SVN if you're migrating to Git.
\*.svn\*
# Some random ones from kernel development, probably not for most people
\*.b
\*.d
\*.asm
{% endhighlight %}

That should cover most files that you don't need in your Git repository.
Also remember to not include files that may contain passwords (or use a
fake password for development), and usually I leave out database files
and instead script the creation of the database with the exact things I
need for testing. This also helps when you need to reset your database.

Aliases
-------

Git provides a way to make aliases in your .gitconfig file. That's all
fine and dandy, but that means I have to do it for each and every
project. Considering almost every bit of work of useful work gets
published in a Git repository, that could get annoying to edit. We could
script it, or we could just make the aliases system wide. I'll go with
system wide, since my bash config files are synced to all my
workstations, and then I can have 2 letter aliases, rather than "git
alias".

**Note:**This is a tip for Ubuntu and should work for Debian. YMMV with
other Linux OS's.

{% highlight bash %}
nano ~/.bash_aliases
--

alias ga="git add -A --ignore-errors" # Add all files in the current directory recursively
alias gc="git commit -a" # Commit changes, will prompt you for commit message
alias gp="git push origin" # Push changes to the server
alias gb="git branch -a" # Show all branches
alias gr="git reset --hard HEAD" # Throw it all away and start with the last commit

# You could do combined commands to add files, commit, and push to the server
alias gacp="git add -A --ignore-errors && git commit -a && git push origin"
{% endhighlight %}

Now when you're at the command line, type any of the above commands,
like "ga" or "gc", and watch the Git magic happen!

[Sensible Git Branching](http://bit.ly/imS419)
----------------------------------------------

Finally, this article lists a bunch of best practices when using Git
branches, especially for open source projects. The described method is
great for developers and great for the users at the same time. It
provides release quality branches, beta quality branches, hotfixes, and
development branches. [This article](http://bit.ly/imS419) is simply a
must-read if you are using Git.
