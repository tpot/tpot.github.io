---
layout: post
title: Using Ccache with Docker
categories: docker
---

<a href="http://ccache.samba.org/">Ccache</a> is a wonderful program
that acts as a front-end for compiling C and C++ files.  If ccache
detects that the same compilation is being done again, it returns the
compilation output from its cache.  The speed increases that are
obtainable from ccache are significant, and it's a very useful program
to know about if you are compiling a lot of C or C++ code.  This article
describes how to use ccache and Docker together to get the isolation
of containers with the speed of a shared compilation cache.


To combine ccache and Docker we're going to use a slightly advanced
concept out of Docker called a <a
href="https://docs.docker.com/userguide/dockervolumes/#data-volumes">data
volume</a>.  Data volumes are a method of decoupling data storage from
the code that's accessing it as well as decoupling the lifetime
of the container performing the compilation from the lifetime of the
cache.  This turns out to be an essential property of our integration.
There's not much point in creating a cache of compiler output if it's
thrown away every time the container running the compilation exits!

Let's start off by creating our data volume in Docker.  In the example
we have some storage set aside on the host filesystem at
<tt>/mnt/ccache</tt> for storing cached data.

Create the data volume called <tt>ccache</tt> using the
following command:

{% highlight bash %}
$ docker create -v /mnt/ccache:/ccache --name ccache debian
{% endhighlight %}

Now let's create a container that "mounts" the data container created
above using the <tt>&#45;&#45;volumes-from</tt> command line option:

{% highlight bash %}
$ docker run -e CCACHE_DIR=/ccache --volumes-from ccache -it debian
{% endhighlight %}

We also pass through the <tt>CCACHE_DIR</tt> environment variable so
that ccache can find our shared cache.  The default cache location is
<tt>$HOME/.ccache</tt> which will be discarded when the container
exits.

Let's give ccache a quick spin by building a small C program to see
whether it works:

{% highlight bash %}
root@827d38c7cc20:/# apt-get update ; apt-get install -y ccache

root@827d38c7cc20:/# cat > foo.c << __EOF__
int main(int argc, char **argv)
{
    return 0;
}
__EOF__

root@827d38c7cc20:/# PATH=/usr/lib/ccache:$PATH gcc -o foo.o -c foo.c
{% endhighlight %}

We can use the <tt>-s</tt> option of ccache to display statistics about the cache:

{% highlight bash %}
root@827d38c7cc20:/# ccache -s
cache directory                     /ccache
cache hit (direct)                     0
cache hit (preprocessed)               0
cache miss                             1
files in cache                         2
cache size                             8 Kbytes
max cache size                       1.0 Gbytes
{% endhighlight %}

We can see from the output above that ccache did see the compilation
and since we have a totally empty cache, a cache miss occured.
Running the compile command again would result in a cache hit and a
(very small in this case) performance increase.  If the cache hit and
miss statistics are all zeros then there is probably something wrong
with your path or the <tt>CCACHE_DIR</tt> environment variable is not
being set correctly.

If you are doing a lot of compilations or have an especially large
project then the default cache size of 1GB may not be large enough.
Disk is cheap these days so bumping that value up to a couple of tens
of gigabytes isn't going to hurt anyone.  Edit the
<tt>/mnt/ccache/ccache.conf</tt> file on the host to do this.