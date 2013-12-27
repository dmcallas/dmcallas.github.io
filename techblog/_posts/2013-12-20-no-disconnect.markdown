---
layout:	post
title:	"Running long commands without worrying about disconnects"
date:	2013-12-20
categories:	Linux SSH
---

Ever run a command on a remote server only to have it fail because
your SSH connection got cut off? Maybe you are doing some batch
processing of data, installing a bunch of packages, or maybe you just
want to leave your IRC reader open after you log off. There are lots
of ways to keep things running after you have left the building.

nohup
=====

When you disconnect from SSH (or any terminal for that matter), any
processes you are running are sent a `SIGHUP` (which is short for
Hangup). Most programs exit when they receive such a signal. If you’d
like to stop your program from receiving the `SIGHUP`, all you need
to do is invoke it with nohup in front. Your program won’t display any
output though; it all is redirected to a file named `nohup.out` (unless
you redirect it somewhere else). That means it is usually best to just
run the command in the background anyways. This usually means you’ll
end up with something like: 

{% highlight bash %}$ nohup bigcommand &{% endhighlight %}

If you’d still like to monitor the progress of the command, you can
always do something like:

{% highlight bash %}$ nohup bigcommand & tail -f nohup.out{% endhighlight %}

The only downside is that this won’t exit when the command finishes,
but at least you can leave the `tail` command at any time without
`bigcommand` terminating.  `nohup` is best for non-interactive commands
you want to have running in the background.


screen
======

`screen` is the go-to command for a lot of people who live on remote
servers. It is useful for a lot more than just keeping commands alive,
but we’ll focus on that bit of it. screen will open a new terminal
that is going to stay alive even if your SSH connection is dropped for
whatever reason. It is perfect if you want to run an interactive
command but don’t want to have a heart attack when your internet drops
out. It isn’t unheard of to keep a long-running irssi session in
screen, for example. To use `screen`, just type `screen` at the bash
prompt and up will pop a new bash prompt within `screen`. If your
connection dies anything you have running will keep running and you
can even log back in and reconnect to your screen session.

If you decide that you’d like to leave `screen` and do something else
while your long-running command is chugging away, just type `Ctrl-a d`
(that is control+`a` followed by the `d` key). That will take you back
to your original bash prompt.

If you either disconnect from screen or get disconnected and have to
log back in, you can see what screen sessions you have going by using

{% highlight bash %}$ screen -ls
There are screens on:
        7540.pts-8.localhost    (Detached)
        30315.pts-5.localhost   (Attached)
2 Sockets in /var/run/screen/S-dmcallas.
{% endhighlight %} 

Any that say “Detached” are not open in a terminal and you can reattach to them. To do that, note the number at the beginning of the screen session (in this case, 7540) and re-attach:

{% highlight bash %}screen -r 7540{% endhighlight %} 

Now you’ll be right back where you left off.
One warning for Emacs users: Most commands in screen begin with Ctrl-a, which in Emacs is a pretty common shortcut key (also useful in bash). If you don’t change the shortcut to something like Ctrl-b, you WILL go insane.

tmux
====

`tmux` is the new kid in town. While it is pretty similar to `screen`, it has quickly gained in popularity. Here is an example of opening `tmux`, disconnecting, and reconnecting:

{% highlight bash %}$ tmux
[detached]
$ tmux list-sessions
0: 1 windows (created Fri Dec 20 23:38:44 2013) [225x64]
$ tmux attach
[exited]
{% endhighlight %} 

Detaching from tmux is accomplished with `Ctrl-b d` similar to `screen` but with `Ctrl-b` instead of `Ctrl-a`.

Conclusion
==========

`screen` and `tmux` both have devoted followers and provide way more
features than I’m mentioning here, but even if this was all they did,
they would be immensely useful tools. If you use remote servers often
you will be well-served by using either one of them. If you only use
them on rare occasions and only have small commands to run, nohup is a
handy and lightweight alternative. In any case, at least you can rest
easy knowing that your programs will no longer depend on your internet
connection to continue running.