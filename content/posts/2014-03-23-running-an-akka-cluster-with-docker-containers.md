---
title: Running an Akka Cluster with Docker Containers
author: Michael
type: post
date: 2014-03-23
url: /2014/03/running-an-akka-cluster-with-docker-containers/
dsq_thread_id:
  - 2493326369
categories:
  - Programming
tags:
  - akka
  - cluster
  - docker
  - scala
---
**Update! You can now use SBT-Docker with SBT-Native Packager for a better sbt/docker experience. [Here&#8217;s the new approach][1] with an [updated GitHub repo][2].**

We recently upgraded our vagrant environments to use [docker][3]. One of our projects relies on [akka&#8217;s cluster functionality][4]. I wanted to easily run an akka cluster locally using docker as sbt can be somewhat tedious. [The example project is on github][2] and the solution is described below.

The solution relies on:

  * [Sbt Native Packager][5] to package dependencies and create a startup file.
  * [Typesafe&#8217;s Config][6] library for configuring the app&#8217;s ip address and seed nodes. We setup cascading configurations that will look for docker link environment variables if present.
  * [A simple bash script][7] to package the app and build the docker container.

I&#8217;m a big fan of [Typesafe&#8217;s Config][8] library and the environment variable overrides come in handy for providing sensible defaults with optional overrides. It&#8217;s the preferred way we configure our applications in upper environments.

The tricky part of running an akka cluster with docker is knowing the ip address each remote node needs to listen on. An akka cluster relies on each node listening on a specific port and hostname or ip. It also needs to know the port and hostname/ip of a seed node the cluster. As there&#8217;s no catch-all binding we need specific ip settings for our cluster.

A simple bash script within the container will figure out the current IP for our cluster configuration and [docker links][9] pass seed node information to newly launched nodes.

## First Step: Setup Application Configuration

The configuration is the same as that of a normal cluster, but I&#8217;m using substitution to configure the ip address, port and seed nodes for the application. For simplicity I setup a `clustering` block with defaults for running normally and environment variable overrides:

<pre class="syntax json">clustering {
 ip = "127.0.0.1"
 ip = ${?CLUSTER_IP}
 port = 1600
 port = ${?CLUSTER_PORT}
 seed-ip = "127.0.0.1"
 seed-ip = ${?CLUSTER_IP}
 seed-ip = ${?SEED_PORT_1600_TCP_ADDR}
 seed-port = 1600
 seed-port = ${?SEED_PORT_1600_TCP_PORT}
 cluster.name = clustering-cluster
}

akka.remote {
    log-remote-lifecycle-events = on
    netty.tcp {
      hostname = ${clustering.ip}
      port = ${clustering.port}
    }
  }
  cluster {
    seed-nodes = [
       "akka.tcp://"${clustering.cluster.name}"@"${clustering.seed-ip}":"${clustering.seed-port}
    ]
    auto-down-unreachable-after = 10s
  }
}
</pre>

As an example the `clustering.seed-ip` setting will use _127.0.0.1_ as the default. If it can find a _CLUSTER_IP_ or a _SEED\_PORT\_1600\_TCP\_ADDR_ override it will use that instead. You&#8217;ll notice the latter override is using docker&#8217;s environment variable pattern for linking: that&#8217;s how we set the cluster&#8217;s seed node when using docker. You don&#8217;t need the _CLUSTER_IP_ in this example but that&#8217;s the environment variable we use in upper environments and I didn&#8217;t want to change our infrastructure to conform to docker&#8217;s pattern. The cascading settings are helpful if you&#8217;re forced to follow one pattern depending on the environment. We do the same thing for the ip and port of the current node when launched.

With this override in place we can use substitution to set the seed nodes in the akka cluster configuration block. The expression `"akka.tcp://"${clustering.cluster.name}"@"${clustering.seed-ip}":"${clustering.seed-port}` builds the proper akka URI so the current node can find the seed node in the cluster. Seed nodes avoid potential split-brain issues during network partitions. You&#8217;ll want to run more than one in production but for local testing one is fine. On a final note the cluster-name setting is arbitrary. Because the name of the actor system and the uri must match I prefer not to hard code values in multiple places.

I put these settings in resources/reference.conf. We could have named this file application.conf, but I prefer bundling configurations as reference.conf and reserving application.conf for external configuration files. A setting in application.conf will override a corresponding reference.conf setting and you probably want to manage application.conf files outside of the project&#8217;s jar file.

## Second: SBT Native Packager

We use the native packager plugin to build a runnable script for our applications. For docker we just need to run `universal:stage`, creating a folder with all dependencies in the `target/` folder of our project. We&#8217;ll move this into a staging directory for uploading to the docker container.

## Third: The Dockerfile and Start script

The dockerfile is pretty simple:

<pre class="syntax bash">FROM dockerfile/java

MAINTAINER Michael Hamrah m@hamrah.com

ADD tmp/ /opt/app/
ADD start /opt/start
RUN chmod +x /opt/start

EXPOSE 1600

ENTRYPOINT [ "/opt/start" ]
</pre>

We start with Dockerfile&#8217;s java base image. We then upload our staging `tmp/` folder which has our application from sbt&#8217;s native packager output and a corresponding executable start script described below. I opted for `ENTRYPOINT` instead of `CMD` so the container is treated like an executable. This makes it easier to pass in command line arguments into the sbt native packager script in case you want to set java system properties or override configuration settings via command line arguments.

The start script is how we tee up the container&#8217;s IP address for our cluster application:

<pre class="syntax bash">#!/bin/bash

CLUSTER_IP=<code>/sbin/ifconfig eth0 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}'</code> /opt/app/bin/clustering $@
</pre>

The script sets an inline environment variable by parsing `ifconfig` output to get the container&#8217;s ip. We then run the _clustering_ start script produced from sbt native packager. The `$@` lets us pass along any command line settings set when launching the container into the sbt native packager script.

## Fourth: Putting It Together

The last part is a simple bash script named `dockerize` to orchestrate each step. By running this script we run sbt native packager, move files to a staging directory, and build the container:

<pre class="syntax bash">#!/bin/bash

echo "Build docker container"

#run sbt native packager
sbt universal:stage

#cleanup stage directory
rm -rf docker/tmp/

#copy output into staging area
cp -r target/universal/stage/ docker/tmp/

#build the container, remove intermediate nodes
docker build -rm -t clustering docker/

#remove staged files
rm -rf docker/tmp/
</pre>

With this in place we simply run

<pre class="brush: plain; title: ; notranslate" title="">bin/dockerize
</pre>

to create our docker container named clustering.

## Running the Application within Docker

With our clustering container built we fire up our first instance. This will be our seed node for other containers:

<pre class="syntax bash">$ docker run -i -t -name seed clustering
2014-03-23 00:20:39,918 INFO  akka.event.slf4j.Slf4jLogger - Slf4jLogger started
2014-03-23 00:20:40,392 DEBUG com.mlh.clustering.ClusterListener - starting up cluster listener...
2014-03-23 00:20:40,403 DEBUG com.mlh.clustering.ClusterListener - Current members:
2014-03-23 00:20:40,418 INFO  com.mlh.clustering.ClusterListener - Leader changed: Some(akka.tcp://clustering-cluster@172.17.0.2:1600)
2014-03-23 00:20:41,404 DEBUG com.mlh.clustering.ClusterListener - Member is Up: akka.tcp://clustering-cluster@172.17.0.2:1600
</pre>

Next we fire up a second node. Because of our reference.conf defaults all we need to do is link this container with the name _seed_. Docker will set the environment variables we are looking for in the bundled reference.conf:

<pre class="syntax bash">$ docker run -name c1 -link seed:seed -i -t clustering
2014-03-23 00:22:49,332 INFO  akka.event.slf4j.Slf4jLogger - Slf4jLogger started
2014-03-23 00:22:49,788 DEBUG com.mlh.clustering.ClusterListener - starting up cluster listener...
2014-03-23 00:22:49,797 DEBUG com.mlh.clustering.ClusterListener - Current members:
2014-03-23 00:22:50,238 DEBUG com.mlh.clustering.ClusterListener - Member is Up: akka.tcp://clustering-cluster@172.17.0.2:1600
2014-03-23 00:22:50,249 INFO  com.mlh.clustering.ClusterListener - Leader changed: Some(akka.tcp://clustering-cluster@172.17.0.2:1600)
2014-03-23 00:22:50,803 DEBUG com.mlh.clustering.ClusterListener - Member is Up: akka.tcp://clustering-cluster@172.17.0.3:1600
</pre>

You&#8217;ll see the current leader discovering new nodes and the appropriate broadcast messages sent out. We can even do this a third time and all nodes will react:

<pre class="syntax bash">$ docker run -name c2 -link seed:seed -i -t clustering
2014-03-23 00:24:52,768 INFO  akka.event.slf4j.Slf4jLogger - Slf4jLogger started
2014-03-23 00:24:53,224 DEBUG com.mlh.clustering.ClusterListener - starting up cluster listener...
2014-03-23 00:24:53,235 DEBUG com.mlh.clustering.ClusterListener - Current members:
2014-03-23 00:24:53,470 DEBUG com.mlh.clustering.ClusterListener - Member is Up: akka.tcp://clustering-cluster@172.17.0.2:1600
2014-03-23 00:24:53,472 DEBUG com.mlh.clustering.ClusterListener - Member is Up: akka.tcp://clustering-cluster@172.17.0.3:1600
2014-03-23 00:24:53,478 INFO  com.mlh.clustering.ClusterListener - Leader changed: Some(akka.tcp://clustering-cluster@172.17.0.2:1600)
2014-03-23 00:24:55,401 DEBUG com.mlh.clustering.ClusterListener - Member is Up: akka.tcp://clustering-cluster@172.17.0.4:1600
</pre>

Try killing a node and see what happens!

## Modifying the Docker Start Script

There&#8217;s another reason for the docker start script: it opens the door for different seed discovery options. Container linking works well if everything is running on the same host but not when running on multiple hosts. Also setting multiple seed nodes via docker links will get tedious via environment variables; it&#8217;s doable but we&#8217;re getting into coding-cruft territory. It would be better to discover seed nodes and set that configuration via command line parameters when launching the app.

The start script gives us control over how we discover information. We could use [etcd][10], [serf][11] or even zookeeper to manage how seed nodes are set and discovered, passing this to our application via environment variables or additional command line parameters. Seed nodes can easily be set via system properties set via the command line:

<pre class="brush: plain; title: ; notranslate" title="">-Dakka.cluster.seed-nodes.0=akka.tcp://ClusterSystem@host1:2552
-Dakka.cluster.seed-nodes.1=akka.tcp://ClusterSystem@host2:2552
</pre>

The start script can probably be configured via sbt native packager but I haven&#8217;t looked into that option. Regardless this approach is (relatively) straight forward to run akka clusters with docker. The [full project is on github][2]. If there&#8217;s a better approach I&#8217;d love to know!

 [1]: http://blog.michaelhamrah.com/2014/06/akka-clustering-with-sbt-docker-and-sbt-native-packager/
 [2]: https://github.com/mhamrah/akka-docker-cluster-example
 [3]: http://docker.io
 [4]: http://doc.akka.io/docs/akka/2.3.0/common/cluster.html
 [5]: https://github.com/sbt/sbt-native-packager
 [6]: https://github.com/typesafehub/config
 [7]: https://github.com/mhamrah/akka-docker-cluster-example/blob/master/bin/dockerize
 [8]: http://blog.michaelhamrah.com/2014/02/leveraging-typesafes-config-library-across-environments/
 [9]: http://docs.docker.io/en/latest/use/working_with_links_names/
 [10]: https://github.com/coreos/etcd
 [11]: http://www.serfdom.io/
