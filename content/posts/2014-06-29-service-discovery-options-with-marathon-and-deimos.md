---
title: Service Discovery Options with Marathon and Deimos
author: Michael
type: post
date: 2014-06-29
url: /2014/06/service-discovery-options-with-marathon-and-deimos/
dsq_thread_id:
  - 2805134320
categories:
  - Programming
tags:
  - docker
  - marathon
  - mesos
---
I&#8217;ve become a fan of Mesos and Marathon: combined with [Deimos][1] you can create a DIY PaaS for launching and scaling Docker containers across a number of nodes. Marathon supports a bare-bones service-discovery mechanism through its task API, but it would be nice for containers to register themselves with some service discovery tool themselves. In order to achieve this containers need to know their host ip address and the port Marathon assigned them so they could tell other interested services where they can be found.

Deimos allows default parameters to be passed in when executing `docker run` and Marathon adds assigned ports to a container&#8217;s environment variables. If a container has this information it can register it with a service discovery tool.

Here we assign the host&#8217;s IP address as a default run option in our [Deimos config file][2].

<pre class="syntax bash">#/etc/deimos.cfg
[containers.options]
append: ["-e", "HOST_IP=192.168.33.12"]
</pre>

Now let&#8217;s launch our mesos-sample container to our Mesos cluster via Marathon:

<pre class="syntax json">// Post to http://192.168.33.12/v2/apps
{
  "container": {
    "image": "docker:///mhamrah/mesos-sample"
  },
  "cpus": "1",
  "id": "www",
  "instances": 1,
  "mem": 512,
  "ports": [0],
  "uris": [],
  "cmd": ""
}
</pre>

Once our app is launch, we can inspect all the environment variables in our container with the `/env` endpoint from `mhamrah/mesos-sample`:

<pre class="syntax json">curl http://192.168.33.12:31894/env
[ {
  "HOSTNAME" : "a4305981619d"
}, {
  "PORT0" : "31894"
}, {
  "PATH" : "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
}, {
  "PWD" : "/tmp/mesos-sandbox"
}, {
  "PORTS" : "31894"
}, {
  "HOST_IP" : "192.168.33.12"
}, {
  "PORT" : "31894"
}]
</pre>

With this information some startup script could use the `PORT` (or `PORT0`) and `HOST_IP` to register itself for direct point-to-point communication in a cluster.

 [1]: https://github.com/mesosphere/deimos
 [2]: https://github.com/mesosphere/deimos/#configuration
