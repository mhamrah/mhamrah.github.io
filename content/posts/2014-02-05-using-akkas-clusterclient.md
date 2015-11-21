---
title: Using Akka’s ClusterClient
author: Michael
type: post
date: 2014-02-05
url: /2014/02/using-akkas-clusterclient/
dsq_thread_id:
  - 2225860500
categories:
  - Programming
tags:
  - akka
  - cluster
  - scala
---
I&#8217;ve been spending some time implementing a feature which leverages Akka&#8217;s [ClusterClient][1] ([api docs][2]). A ClusterClient can be useful if:

  * You are running a service which needs to talk to another service in a cluster, but you don&#8217;t that service to be in the cluster (cluster roles are another option for interconnecting services where separate hosts are necessary but I&#8217;m not sold on them just yet).
  * You don&#8217;t want the overhead of running an http client/server interaction model between these services, but you&#8217;d like similar semantics. Spray is a great akka framework for api services but you may not want to write a Spray API or use an http client library.
  * You want to use the same transparency of local-to-remote actors but don&#8217;t want to deal with remote actorref configurations to specific hosts.

The documentation was a little thin on some specifics so getting started wasn&#8217;t as smooth sailing as I&#8217;d like. Here are some gotchas:

  * You only need `akka.extensions = ["akka.contrib.pattern.ClusterReceptionistExtension"]` on the Host (Server) Cluster. (If your client isn&#8217;t a cluster, you&#8217;ll get a runtime exception).
  * &#8220;Receptionist&#8221; is the default name for the Host Cluster actor managing ClusterClient connections. Your ClusterClient connects first to the receptionist (via the set of initial contacts) then can start sending messages to actors in the Host Cluster. The name is configurable.
  * The client actor system using the ClusterClient needs to have a Netty port open. You must use either actor.cluster.ClusterActorRefProvider or actor.remote.RemoteActorRefProvider. Otherwise the Host Cluster and ClusterClient can&#8217;t establish proper communication. You can use the ClusterActorRefProvider on the client even you&#8217;re not running a cluster. 
  * As a ClusterClient you wrap messages with a ClusterClient.send (or sendAll) message first. (I was sending vanilla messages and they weren&#8217;t going through, but this is in the docs).

ClusterClients are worth checking out if you want to create physically separate yet interconnected systems but don&#8217;t want to go through the whole load-balancer or http-layer setup. Just another tool in the Akka toolbelt!

 [1]: http://doc.akka.io/docs/akka/2.2.3/contrib/cluster-client.html
 [2]: http://doc.akka.io/api/akka/2.2.3/index.html#akka.contrib.pattern.ClusterClient
