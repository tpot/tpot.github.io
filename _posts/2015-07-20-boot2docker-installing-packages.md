---
layout: post
title: Installing Extra Packages on Boot2docker
categories: docker
---

Like a lot of other Mac users, I run <a
href="http://boot2docker.io">boot2docker</a> to run Docker on my
laptop.  Boot2docker is a tiny Linux VM that is usually run headless
under Oracle's VirtualBox.  Boot2docker runs a relatively unknown
Linux distribution called <a href="http://tinycorelinux.net/">Tiny
Core Linux</a>.  Tiny Core is pretty neat, but it's not obvious how to
install extra packages like you might easily do on other Linux
distributions.  Here's how to install extra packages on your
boot2docker instances.


The first step is to figure out whether the package you want is
already been packaged.  To do this check the <a
href="http://distro.ibiblio.org/tinycorelinux/tcz_2x.html">Tiny Core
Linux software repository</a> page where there are currently a couple
of hundred packages already made.  I often need to copy files to and
from a boot2docker instance and the easiest way to do that is using
rsync, so this what I'll show being installed.

The second step is to determine the name of the package binary from
the repository web page and install it using the package install tool.
Unlike larger Linux distributions, Tiny Core Linux does not have a
searchable package database so you will need to figure out the package
name.  Just choosing $PACKAGENAME.tcz is going to be a good guess.
The package name for rsync is <tt>rsync.tcz</tt> so let's install
that using <tt>tce-load</tt>:

{% highlight bash %}
docker@boot2docker:~$ tce-load -w -i rsync.tcz
Connecting to repo.tinycorelinux.net (89.22.99.37:80)
popt.tcz             100% |*******************************| 28672   0:00:00 ETA
popt.tcz: OK
Downloading: rsync.tcz
Connecting to repo.tinycorelinux.net (89.22.99.37:80)
rsync.tcz            100% |*******************************|   180k  0:00:00 ETA
rsync.tcz: OK
{% endhighlight %}

Finally, let's test rsync has been installed:

{% highlight bash %}
docker@boot2docker:~$ rsync
rsync  version 3.0.0  protocol version 30
Copyright (C) 1996-2007 by Andrew Tridgell, Wayne Davison, and others.
Web site: http://rsync.samba.org/
Capabilities:
    64-bit files, 64-bit inums, 32-bit timestamps, 64-bit long ints,
    socketpairs, hardlinks, symlinks, IPv6, batchfiles, inplace,
    append, no ACLs, xattrs, iconv, no symtimes
{% endhighlight %}

Rsync has been successfully installed on your boot2docker instance.