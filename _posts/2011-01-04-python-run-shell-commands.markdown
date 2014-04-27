---
title: Run Shell Commands in Python


date: 2011-01-04 15:00

author: Josh

category: Articles

tags: Python

permalink: python-run-shell-commands
---

A lot of my scripts need a simple way to run shell commands. So I went
ahead and wrote a method that makes it super easy to execute commands,
and keep the output of both STDOUT and STDERR logged. Basically, just
build a tuple of commands, such as:

{% highlight python %}
args = ["ls", "-l"]
{% endhighlight %}

This builds a command to be executed, starting with the executable "ls"
(which lists the contents of the current directory), and the argument
"-l" (which lists extra information about the contents).

You can execute the script at the bottom like this:

{% highlight python %}
python shell.py ls -l
{% endhighlight %}

And it will execute just like running the normal "ls -l" command, except
it will redirect all output to two files in the current directory, named
output.log and error.log.

{% highlight python %}
import os, subprocess, sys

def main(argv):
    run(argv)

def run(arg):
    """
    Runs the given arg at the command line using the default shell.
    Outputs when commands are run successfully.

    Based on http://developer.spikesource.com/wiki/index.php/How_to_invoke_subprocesses_from_Python

    @param Tuple args
        A tuple of args, with the first being the command to be run, and
        the remaining ones flags and arguments for the command. STDOUT and
        STDERR are piped to tuple, waiting until the output is finished,
        then writing both to the log files, if not empty.
        Ex. ['apt-get', '-y', 'install', 'dnsmasq'], which installs
        dnsmasq using apt-get, and assumes yes to questions.
        """

    # Open output and write some info about the command to be written, including
    # name of command and arguments.
    # This could be modified to adjust how much is printed via a DEBUG variable.
    with open(os.path.join(os.curdir, "output.log"), 'a') as outFile:
        outFile.write("Command: ")
    for a in arg:
        outFile.write(a,)
        outFile.write(" ")
        outFile.write("\n")
    # Open output and error log file and append to them the output of the commands
    with open(os.path.join(os.curdir, "output.log"), 'a') as outFile:
        with open(os.path.join(os.curdir, "error.log"), 'a') as errorFile:
            # Call the subprocess using convenience method

            retval = subprocess.call(arg, -1, None, None, outFile, errorFile)
            # Check the process exit code, print error information if it exists
            if not retval == 0:
                errData = errorFile.read()
                raise Exception("Error executing command: " + repr(errData))

if __name__=="__main__":
    main(sys.argv[1:])
{% endhighlight %}

If you want to use it in your own scripts, just insert the run method,
and pass the appropriate arguments to it. One potential change that
might be helpful would be to move the log files to /var/log with the
rest of the Linux logs. However normal users don't usually have write
permissions in /var/log, so an install wrapper would be helpful to both
make the 2 files with writable permissions, and to move the file to
/usr/bin so it can more easily be executed from the command line.
