---
title: Check if script is run as root in Bash

date: 2010-11-08 20:53

author: Josh

layout: default
category: Articles

tags: Bash

permalink:  bash-check-if-script-is-run-as-root
---
Many scripts need to be run as root, such as ones installing software,
working with the root filesystem, etc. Instead of having your script run
and many commands fail, its best to check at the beginning whether the
script is being run as root or not. If not, it should exit immediately.

{% highlight bash %}
function sanity_check {
if [ "(id -u)" !="0"; ]; then
echo “Script must be run as root.”
exit 1
fi
}
{% endhighlight %}

This script uses the id command, which prints out all the id’s of the
current user (user id, group id, etc). The -u flag specifies user id.
Root is always user id 0. If the current users’ id is not 0, they must
not be root, therefore, the script tells the user of the problem, then
gracefully exits with exit code 1 (a customary way to say “expected
error that requires exiting”).
