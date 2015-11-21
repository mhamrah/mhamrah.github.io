---
title: Accessing the Docker Host Server Within a Container
author: Michael
type: post
date: 2014-06-29
url: /2014/06/accessing-the-docker-host-server-within-a-container/
dsq_thread_id:
  - 2805133792
categories:
  - Programming
tags:
  - devops
  - docker
  - marathon
  - mesos
---
[Docker links][1] are a great way to link two containers together but sometimes you want to know more about the host and network from within a container. You have a couple of options:

  * You can access the Docker host by the container&#8217;s gateway.
  * You can access the Docker host by its ip address from within a container.

## The Gateway Approach

[This GitHub Issue][2] outlines the solution. Essentially you&#8217;re using netstat to parse the gateway the docker container uses to access the outside world. This is the docker0 bridge on the host.

As an example, we&#8217;ll run a simple docker container which returns the hostname of the container on port 8080:

<pre class="syntax bash">docker run -d -p 8080:8080 mhamrah/mesos-sample
</pre>

Next we&#8217;ll run /bin/bash in another container to do some discovery:

<pre class="syntax bash">docker run -i -t ubuntu /bin/bash
#once in, install curl:
apt-get update
apt-get install -y curl
</pre>

We can use the following command to pull out the gateway from netstat:

<pre class="syntax bash">netstat -nr | grep '^0\.0\.0\.0' | awk '{print $2}'
#returns 172.17.42.1 for me.
</pre>

We can then curl our other docker container, and we should get that docker container&#8217;s hostname:

<pre class="syntax bash">curl 172.17.42.1:8080
# returns 00b019ce188c
</pre>

Nothing exciting, but you get the picture: it doesn&#8217;t matter that the service is inside another container, we&#8217;re accessing it via the host, and we didn&#8217;t need to use links. We just needed to know the port the other service was listening on. If you had a service running on some other port&#8211;say Postgres on 5432&#8211;not running in a Docker container&#8211;you can access it via `172.17.42.1:5432`.

If you have docker installed in your container you can also query the docker host:

<pre class="syntax bash"># In a container with docker installed list other containers running on the host for other containers:
docker -H tcp://172.17.42.1:2375 ps
CONTAINER ID        IMAGE                         COMMAND                CREATED              STATUS              PORTS                     NAMES
09d035054988        ubuntu:14.04                  /bin/bash              About a minute ago   Up About a minute   0.0.0.0:49153->8080/tcp   angry_bardeen
00b019ce188c        mhamrah/mesos-sample:latest   /opt/delivery/bin/de   8 minutes ago        Up 8 minutes        0.0.0.0:8080->8080/tcp    suspicious_colden
</pre>

You can use this for some hakky service-discovery.

## The IP Approach

The gateway approach is great because you can figure out a way to access a host from entirely within a container. You also have the same access via the host&#8217;s ip address. I&#8217;m using boot2docker, and the boot2docker ip address is `192.168.59.103` and I can accomplish the same tasks as the gateway approach:

<pre class="syntax bash"># Docker processes, via ip:
docker -H tcp://192.168.59.103:2375 ps
# Other docker containers, via ip:
curl 192.168.59.103:8080
</pre>

Although there&#8217;s no way to introspect the host&#8217;s ip address (AFAIK) you can pass this in via an environment variable:

<pre class="syntax bash">docker@boot2docker:~$  docker run -i -t -e DOCKER_HOST=192.168.59.103 ubuntu /bin/bash
root@07561b0607f4:/# env
HOSTNAME=07561b0607f4
DOCKER_HOST=192.168.59.103
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/
</pre>

If the container knows the ip address of its host, you can broadcast this out to other services via the container&#8217;s application. Useful for service discovery tools run from within a container where you want to tell others the host IP so others can find you.

 [1]: https://docs.docker.com/userguide/dockerlinks/#working-with-links-names
 [2]: https://github.com/dotcloud/docker/issues/1143
