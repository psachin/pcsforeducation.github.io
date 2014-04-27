---
title: Migrate Git Project from GitHub to Google Code


date: 2011-07-16 12:46

author: Josh

category: Articles

tags: Git, GitHub, Google, Google Code

permalink: migrate-git-project-from-github-to-google-code
---

Today, Google announced they were adding Git support to Google Code.
This is certainly the most requested feature, and one I've been
patiently waiting on. I move from Google Code's SVN repositories to
GitHub's Git repositories almost 2 years ago. I love GitHub, but
frankly, I'd rather have one less account to worry about and move over
to Google Code with a few projects. Both have their advantages, and I'm
not completely entirely convinced Google Code is better, but we will
see.

As for the comparison between GitHub and Google Code, I miss the ability
to follow projects like you can in GitHub. However, Google Code provides
many methods to follow a project in "Project Feeds". They can be found
at <http://code.google.com/p/PROJECT/feeds>, such as Atom feeds of
project updates, wiki updates, issues, source changes, etc. This allows
you to integrate the updates into your existing tools, such as Google
Reader, rather than going to the GitHub website. I will probably keep my
account just for that. So far, I haven't found the same functionality in
Google Code. The wikis seems about the same. Google code does have a
similar function to GitHub's forking and pull request with clone and
code review. Google seems to have integrated their Gerrit project for
code review, which I've heard is fantastic for merging code, and is used
extensively for the Android project, which is also run on Git. Google
Code does provide 4GB of space per free project, where as GitHub only
provides 300MB, though that can be adjusted with an email to their
support. GitHub has a way to download the current source as a .zip or
.tar.gz, which Google seems to be lacking. Google Code does have a way
to script your uploads, which I like very much. They provide an easy way
to use Python to upload files for download. I think this is handy,
because I have always wished for a build server to work with GitHub more
easily. This could easily be used to build .deb or .rpm packages and
have them uploaded and downloaded by users. There are some methods to do
this with GitHub, but none provided by the company themselves.

We are going to copy up all the history and branches from our projects.
If we just copied the files into the new cloned Google repository, we'd
only have the files, and no history. First, head over to[create a new
Google Code project](http://code.google.com/hosting/createProject).
Create your project as normal, selecting Git as your version control
system. Now, head to the terminal, so we can avoid typing in our
password all the time.

{% highlight bash %}
nano ~/.netrc
-----

machine code.google.com login YOUR_LOGIN password YOUR_PASSWORD
-----
chmod 500 ~/.netrc
{% endhighlight %}

This will create a file that is used each time you try to interact with
Google's Git server, containing your login details. By chmod'ing it, no
one but you should be able to read or write it.

Now head into your existing GitHub repository. If you don't have it
locally on your machine, just clone the repository like so:

{% highlight bash %}
git clone git@github.com:username/example_project.git
{% endhighlight %}

We are going to add the Google server as a new repository, and from
there we will clone each branch into the Google repository. This is kind
of a pain, but I did not see an easier way to clean each branch.
Hopefully someone can prove me wrong.

{% highlight bash %}
# Move into the Git directory we just cloned.
cd example_project
# I'm going to call the new repository "google". I never much liked "origin".
git remote add google https://code.google.com/p/example_project/
# Push all branches to the google repository
git push google --all

# Optional: Remove old repo, assuming your GitHub repo is called origin
git remote rm origin
# If you're hung up on the name "origin" you can rename the repo
nano .git/config
-----
# Change:
[remote "google"]
# To:
[remote "origin"]
{% endhighlight %}

That's everything. If you want a more in depth look at how to use Git,
check out my [Git
Guide](http://www.servercobra.com/the-only-guide-to-git-youll-ever-need/),
which has a bunch of shortcuts and a link to an even better article.
