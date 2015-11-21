---
title: Clustering Akka Applications with Docker — Version 3
author: Michael
type: post
date: 2014-11-27
url: /2014/11/clustering-akka-applications-with-docker-version-3/
dsq_thread_id:
  - 3266547318
categories:
  - Programming
tags:
  - akka
  - clustering
  - sbt
  - scala
---
The SBT Native Packager plugin now offers first-class Docker support for building Scala based applications. My last post involved combining SBT Native Packager, SBT Docker, and a custom start script to launch our application. We can simplify the process in two ways:

  1. Although the SBT Docker plugin allows for better customization of Dockerfiles it&#8217;s unnecessary for our use case. SBT Native Packager is enough.
  2. A separate start script was required for IP address inspection so TCP traffic can be routed to the actor system. I recently contributed an update for [better ENTRYPOINT support within SBT Native Packager][1] which gives us options for launching our app in a container.

With this PR we can now add our IP address inspection snippet to our build removing the need for extraneous files. We could have added this snippet to `bashScriptExtraDefines` but that is a global change, requiring `/sbin/ifconfig eth0` to be available wherever the application is run. This is definitely infrastructure bleed-out and must be avoided.

[The new code, on GitHub,][2] uses a shell with ENTRYPOINT exec mode to set our environment variable before launching the application:

<pre class="code scala">dockerExposedPorts in Docker := Seq(1600)

dockerEntrypoint in Docker := Seq("sh", "-c", "CLUSTER_IP=`/sbin/ifconfig eth0 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1 }'` bin/clustering $*")
</pre>

The `$*` allows for command-line parameters to be honored when launching the container. Because the app leverages the Typesafe Config library we can also set via Java system properties:

<pre class="code bash">docker run --rm -i -t --name seed mhamrah/clustering:0.3 -Dclustering.cluster.name=example-cluster
</pre>

Launching the cluster is exactly as before:

<pre class="code bash">docker run --rm -d --name seed mhamrah/clustering:0.3
docker run --rm -d --name member1 --link seed:seed mhamrah/clustering:0.3
</pre>

For complex scripts it may be too messy to overload the ENTRYPOINT sequence. For those cases simply bake your own docker container as a base and use the ENTRYPOINT approach to call out to your script. SBT Native Packager will still upload all your dependencies and its bash script to `/opt/docker/bin/<your app>`. The Docker `WORKDIR` is set to `/opt/docker` so you can drop the `/opt/docker` as above.

 [1]: https://github.com/sbt/sbt-native-packager/pull/411
 [2]: https://github.com/mhamrah/akka-docker-cluster-example
