---
layout: post
title: Exposing a HTTP endpoint for Docker
categories: docker
---

Recent versions of Docker (>= 1.3.0) no longer expose a HTTP endpoint, opting instead the more secure HTTPS protocol.  Unfortunately it may not be possible in some environments or with some tools to use HTTPS.  The following is a quick workaround to use the <tt>socat</tt> utility to proxy access to the Docker socket.


My particular use case is the <a href="https://wiki.jenkins-ci.org/display/JENKINS/Docker+Plugin">Jenkins Docker Plugin</a>.  Since Jenkins is Java-based, SSL issues require messing with the Java Trust Store.  The managment of certificates and trust configuration in Java is decidely un-Unixy and not really something I want to get involved in.  Luckly there is a Docker container available to solve the problem of the missing HTTP endpoint.

Execute the commands below to pull a copy of the <tt>sequenceiq/socat</tt> container and run it, opening up TCP port 2375, the original Docker HTTP port.  This should allow clients to avoid having to use HTTPS to access Docker.

{% highlight bash %}
$ docker pull sequenceiq/socat
$ docker run -d \
      -p 2375:2375 \
      --volume=/var/run/docker.sock:/var/run/docker.sock \
      --name=docker-http \
      sequenceiq/socat
{% endhighlight %}

The <tt>sequenceiq/socat</tt> container is documented more fully in this <a href="http://blog.sequenceiq.com/blog/2014/10/17/boot2docker-tls-workaround/">blog post</a> at the SequenceIQ website.