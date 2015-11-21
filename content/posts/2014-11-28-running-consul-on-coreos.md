---
title: Running Consul on CoreOS
author: Michael
type: post
date: 2014-11-29
url: /2014/11/running-consul-on-coreos/
dsq_thread_id:
  - 3272589494
categories:
  - Programming
tags:
  - clustering
  - consul
  - coreos
  - docker
---
I&#8217;m a big fan of [Consul][1], Hashicorp&#8217;s service discovery tool. I&#8217;ve also become a fan of CoreOS, the cluster framework for running docker containers. Even though CoreOS comes with etcd for service discovery I find the feature set of Consul more compelling. And as a programmer I know I can have my cake and eat it too.

My first take was to modify my [ansible-consul][2] fork to run consul natively on CoreOS. Although this could work I find it defeats CoreOS&#8217;s container-first approach with [fleet][3]. Jeff Lindsay created [a consul docker container][4] which does the job well. I created two fleet service files: one for launching the consul container and another for service discovery. At first the service discovery aspect seemed weird; I tried to pass ip addresses via the &#8211;join parameter or use `ExecStartPost` for running the join command. However I took a cue from the CoreOS cluster setup: sometimes you need a third party to get stuff done. In this case we the built in etcd server to manage the join ip address to kickstart the consul cluster.

The second fleet service file acts as a sidekick:

  * For every running consul service there&#8217;s a sidekick process
  * The sidekick process writes the current IP to a key only if that key doesn&#8217;t exist
  * The sidekick process uses the value of that key to join the cluster with `docker exec`
  * The sidekick process removes the key if the consul service dies. 

The two service files are below, but you should tweak for your needs.

  * You only need a 3 or 5 node server cluster. If your CoreOS deployment is large, use some form of restriction for the server nodes. You can do the same for the client nodes.
  * The discovery script could be optimized. It will try and join whatever ip address is listed in the key. This avoids a few split brain scenarios, but needs to be tested.
  * If you want DNS to work properly you need to set some Docker daemon options. Read the docker-consul README.

 [1]: consul.io
 [2]: github.com/mhamrah/ansible-consul
 [3]: https://github.com/coreos/fleet
 [4]: https://github.com/progrium/docker-consul
