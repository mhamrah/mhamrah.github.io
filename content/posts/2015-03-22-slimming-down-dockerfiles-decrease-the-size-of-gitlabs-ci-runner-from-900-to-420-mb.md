---
title: 'Slimming down Dockerfiles: Decrease the size of Gitlab’s CI runner from 900 to 420 mb'
author: Michael
type: post
date: 2015-03-22
url: /2015/03/slimming-down-dockerfiles-decrease-the-size-of-gitlabs-ci-runner-from-900-to-420-mb/
dsq_thread_id:
  - 3616921021
categories:
  - Programming
tags:
  - docker
  - dockerfile
  - linux
---
I&#8217;ve been leveraging Gitlab CI for our continuous integration needs, running both the CI site and CI runners on our CoreOS cluster in docker containers. It&#8217;s working well. On the runner side, after cloning the ci-runner repositroy and running a `docker build -t base-runner .` , I was a little disappointed with the size of the runner. It weighed in at 900MB, a fairly hefty size for something that should be a lightweight process. I&#8217;ve built the [ci-runner dockerfile][1] with the name &#8220;base-runner&#8221;:

<pre class="syntax bash wrap:true">base-runner      latest      aaf8a1c6a6b8    2 weeks ago    901.1 MB
</pre>

The dockerfile is well documented and organized, but I immediately noticed some things which cause dockerfile bloat. There are some great resources on slimming down docker files, including [optimizing docker images][2] and the [docker-alpine][3] project. The advice comes down to:

  * Use the smallest possible base layer (usually Ubuntu is not needed)
  * Eliminate, or at least reduce, layers
  * Avoid extraneous cruft, usually due to excessive packages

Let&#8217;s make some minor changes to see if we can slim down this image. At the top of the dockerfile, we see the usual apt-get commands:

<pre class="syntax bash wrap:true"># Update your packages and install the ones that are needed to compile Ruby
RUN apt-get update -y
RUN apt-get upgrade -y
RUN apt-get install -y curl libxml2-dev libxslt-dev libcurl4-openssl-dev libreadline6-dev libssl-dev patch build-essential zlib1g-dev openssh-server libyaml-dev libicu-dev

# Download Ruby and compile it
RUN mkdir /tmp/ruby
RUN cd /tmp/ruby &#038;&#038; curl --silent ftp://ftp.ruby-lang.org/pub/ruby/2.0/ruby-2.0.0-p481.tar.gz | tar xz
RUN cd /tmp/ruby/ruby-2.0.0-p481 &#038;&#038; ./configure --disable-install-rdoc &#038;&#038; make install
</pre>

Each `RUN` command creates a separate layer, and nothing is cleaned up. These artifacts will stay with the container unnecessarily. Running another `RUN rm -rf /tmp` won&#8217;t help, because the history is still there. We need things gone and without a trace. We can &#8220;flatten&#8221; these commands and add some cleanup commands while preserving readability:

<pre class="syntax bash"># Update your packages and install the ones that are needed to compile Ruby
# Download Ruby and compile it
RUN apt-get update -y &#038;&#038; 
    apt-get upgrade -y &#038;&#038; 
    apt-get install -y curl libxml2-dev libxslt-dev libcurl4-openssl-dev libreadline6-dev libssl-dev patch build-essential zlib1g-dev openssh-server libyaml-dev libicu-dev &#038;&#038; 
    mkdir /tmp/ruby &#038;&#038; 
    cd /tmp/ruby &#038;&#038; curl --silent ftp://ftp.ruby-lang.org/pub/ruby/2.0/ruby-2.0.0-p481.tar.gz | tar xz &#038;&#038; 
    cd /tmp/ruby/ruby-2.0.0-p481 &#038;&#038; ./configure --disable-install-rdoc &#038;&#038; make install &#038;&#038; 
    apt-get clean &#038;&#038; 
    rm -rf /var/lib/apt/lists/* /tmp/*
</pre>

There&#8217;s only one run command, and the last two lines cleanup the apt-get downloads and `tmp` space. Let&#8217;s see how we well we do:

<pre class="syntax bash wrap:true">$ docker images | grep base-runner
base-runner      latest      2a454f84e4e8      About a minute ago   566.9 MB
</pre>

Not bad; with one simple modification we went from 902mb to 566mb. This change comes at the cost of build speed. Because there&#8217;s no previously cached layer, we always start from the beginning. When creating docker files, I usually start with multiple run commands so history is preserved while I&#8217;m working on the file, but then concatenate everything at the end to minimize cruft.

566mb is a good start, but can we do better? The goal of this build is to install the ci-runner. This requires Ruby and some dependencies, all documented on the ci-runner&#8217;s readme. As long as we&#8217;re meeting those requirements, we&#8217;re good to go. Let&#8217;s switch to debian:wheezy. We&#8217;ll also need to tweak the locale setting for debian. Our updated dockerfile starts with this:

<pre class="syntax bash wrap:true"># gitlab-ci-runner

FROM debian:wheezy
MAINTAINER Michael Hamrah &lt;m@hamrah.com>

# Get rid of the debconf messages
ENV DEBIAN_FRONTEND noninteractive

# Update your packages and install the ones that are needed to compile Ruby
# Download Ruby and compile it
RUN apt-get update -y &#038;&#038; 
    apt-get upgrade -y &#038;&#038; 
    apt-get install -y locales curl libxml2-dev libxslt-dev libcurl4-openssl-dev libreadline6-dev libssl-dev patch build-essential zlib1g-dev openssh-server libyaml-dev libicu-dev &#038;&#038; 
    mkdir /tmp/ruby &#038;&#038; 
    cd /tmp/ruby &#038;&#038; curl --silent ftp://ftp.ruby-lang.org/pub/ruby/2.0/ruby-2.0.0-p481.tar.gz | tar xz &#038;&#038; 
    cd /tmp/ruby/ruby-2.0.0-p481 &#038;&#038; ./configure --disable-install-rdoc &#038;&#038; make install &#038;&#038; 
    apt-get clean &#038;&#038; 
    rm -rf /var/lib/apt/lists/* /tmp/*
</pre>

Let&#8217;s check this switch:

<pre class="syntax bash wrap:true">base-runner      latest      40a1465ebaed      3 minutes ago      490.3 MB
</pre>

Better. A slight modification can slim this down some more; the dockerfile builds ruby from source. Not only does this take longer, it&#8217;s not needed: we can just include the `ruby` and `ruby-dev` packages; on debian:wheezy these are good enough for running the ci-runner. By removing the install-from-source commands we can get the image down to:

<pre class="syntax bash wrap:true">base-runner      latest      bb4e6306811d      About a minute ago   423.6 MB
</pre>

This now more than 50% less then the original, with a minimal amount of tweaking.

# Pushing Even Further

Normally I&#8217;m not looking for an absolute minimal container. I just want to avoid excessive bloat, and some simple commands can usually go a long way. I also find it best to avoid packages in favor of pre-built binaries. As an example I do a lot of work with Scala, and have an sbt container for builds. If I were to install the SBT package from debian I&#8217;d get a container weighing in at a few hundred megabytes. That&#8217;s because the SBT package pulls in a lot of dependencies: java, for one. But if I already have a jre, all I really need is the sbt jar file and a bash script to launch. That considerably shrinks down the dockerfile size.

When selecting a base image, it&#8217;s important to realize what you&#8217;re getting. A linux distribution is simply the linux kernel and an opinionated configuration of packages, tools and binaries. Ubuntu uses aptitude for package management, Fedora uses Yum. Centos 6 uses a specific version of the kernel, while version 7 uses another. You get one set of packages with Debian, another with Ubuntu. That&#8217;s the power of specific communities: how frequently things are updated, how well they&#8217;re maintained, and what you get out-of-box. A docker container jails a process to only see a specific part of the filesystem; specifically, the container&#8217;s file system. Using a distribution ensures that the required libraries and support binaries are there, in place, where they should be. But major distributions aren&#8217;t designed to run specific applications; their general-purposes servers that are designed to run a variety of apps and processes. If you want to run a single process there&#8217;s a lot that comes with an OS you don&#8217;t need.

There&#8217;s a re-emergence of lightweight linux distributions in the docker world first popularized with embedded systems. You&#8217;re probably familiar with [busybox][4] useful for running one-off bash commands. Because of busybox&#8217;s embedded roots, it&#8217;s not quite intended for packages. [Alpine Linux][5] is another alternative which features its own [official registry][6]. It&#8217;s still very small, based on busybox, and has its own package system. I tried getting gitlab&#8217;s ci-runner working with alpine, but unfortunately some of the ruby gems used by ci-runner require GNU packages which aren&#8217;t available, and I didn&#8217;t want to compile them manually. In terms of time/benefit, I can live with 400mb and move on to something else. For most things you can probably do a lot with Alpine and keep your containers really small: great for doing continuous deploys to a bunch of servers.

The bottom line is know what you need. If you want a minimal container, build up, rather than slim down. You usually need the runtime for your application (if any), your app, and supporting dependencies. Know those dependencies, and avoid cruft when you can.

 [1]: https://github.com/gitlabhq/gitlab-ci-runner/blob/master/Dockerfile
 [2]: http://www.centurylinklabs.com/optimizing-docker-images/
 [3]: https://github.com/gliderlabs/docker-alpine
 [4]: https://registry.hub.docker.com/_/busybox/
 [5]: https://www.alpinelinux.org
 [6]: https://registry.hub.docker.com/_/alpine/
