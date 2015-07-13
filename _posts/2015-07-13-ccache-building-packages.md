---
layout: post
title: Building Debian Packages with Ccache
categories: debian
---

Following on from a previous post about <a href="{% post_url 2015-07-09-using-ccache-with-docker %}">
Using Ccache with Docker</a>, here's how to build a Debian packaging
using <a href="http://cache.samba.org/">Ccache</a>.


I've been building a lot of Debian packages recently and Ccache has
been instrumental in making rebuilds incredibly fast.  Here's the
command I use to rebuild a package without having to mess around and
actually modify the package at all:

{% highlight bash %}
$ debuild --preserve-envvar=CCACHE_DIR \
    --prepend-path=/usr/lib/ccache \
    -us -uc
{% endhighlight %}

Note that both the <tt>--preserve-envvar</tt> and
<tt>--prepend-path</tt> options must be before <tt>-us</tt> and
<tt>-uc</tt>.  The former are debuild options and he latter
dpkg-buildpackage options.
