---
title: Setting up a Multi-Node Mesos Cluster running Docker, HAProxy and Marathon with Ansible
author: Michael
type: post
date: 2014-06-26
url: /2014/06/setting-up-a-multi-node-mesos-cluster-running-docker-haproxy-and-marathon-with-ansible/
dsq_thread_id:
  - 2795741319
categories:
  - Programming
tags:
  - ansible
  - devops
  - marathon
  - mesos
---
**UPDATE**
  
_With Mesos 0.20 Docker support is now native, and Deimos has been deprecated. The ansible-mesos-playbook has been updated appropriately, and most of this blog post still holds true. There are slight variations with how you post to Marathon._

The [Google Omega Paper][1] has given birth to cloud vNext: cluster schedulers managing containers. You can make a bunch of nodes appear as one big computer and deploy anything to your own private cloud; just like Docker, but across any number of nodes. [Google&#8217;s Kubernetes][2], [Flynn][3], [Fleet][4] and [Apache Mesos][5], originally from Twitter, are implementations of Omega with the goal of abstracting away discrete nodes and optimizing compute resources. Each implementation has its own tweak, but they all follow the same basic setup: leaders, for coordination and scheduling; some service discovery component; some underlying cluster tool (like Zookeeper); followers, for processing.

In this post we&#8217;ll use [Ansible][6] to install a multi-node Mesos cluster using packages from [Mesosphere][7]. Mesos, as a cluster framework, allows you to run a variety of cluster-enabled software, including [Spark][8], [Storm][9] and [Hadoop][10]. You can also run [Jenkins][11], schedule tasks with [Chronos][12], even run [ElasticSearch][13] and [Cassandra][14] without having to double to specific servers. We&#8217;ll also set up [Marathon][15] for running services with [Deimos][16] support for Docker containers.

Mesos, even with Marathon, doesn&#8217;t offer the holistic integration of some other tools, namely Kubernetes, but at this point it&#8217;s easier to set up on your own set of servers. Although young Mesos is one of the oldest projects of the group and allows more of a DIY approach on service composition.

### TL;DR

_[The playbook is on github, just follow the readme!][17]_. If you want to simply try out Mesos, Marathon, and Docker [mesosphere has an excellent tutorial to get you started on a single node][18]. This tutorial outlines the creation of a more complex multi-node setup.

### System Setup

The system is divided into two parts: a set of masters, which handle scheduling and task distribution, with a set of slaves providing compute power. Mesos uses Zookeeper for cluster coordination and leader election. A key component is service discovery: you don&#8217;t know which host or port will be assigned to a task, which makes, say, accessing a website running on a slave difficult. The Marathon API allows you to query task information, and we use this feature to configure HAProxy frontend/backend resources.

Our masters run:

  * Zookeeper
  * Mesos-Master
  * HAProxy
  * Marathon

and our slaves run:

  * Mesos-Slave
  * Docker
  * Deimos, the Mesos -> Docker bridge

### Ansible

Ansible works by running a playbook, composed of roles, against a set of hosts, organized into groups. My [Ansible-Mesos-Playbook][17] on GitHub has an example hosts file with some EC2 instances listed. You should be able to replace these with your own EC2 instances running Ubuntu 14.04, our your own private instances running Ubuntu 14.04. Ansible allows us to pass node information around so we can configure multiple servers to properly set up our masters, zookeeper set, point slaves to masters, and configure Marathon for high availability.

We want at least three servers in our master group for a proper zookeeper quorum. We use host variables to specify the zookeeper id for each node.

<pre class="brush: plain; title: ; notranslate" title="">[mesos_masters]
ec2-54-204-214-172.compute-1.amazonaws.com zoo_id=1
ec2-54-235-59-210.compute-1.amazonaws.com zoo_id=2
ec2-54-83-161-83.compute-1.amazonaws.com zoo_id=3
</pre>

The [mesos-ansible][19] playbook will use nodes in the `mesos_masters` for a variety of configuration options. First, the `/etc/zookeeper/conf/zoo.cfg` will list all master nodes, with `/etc/zookeeper/conf/myid` being set appropriately. It will also set up upstart scripts in `/etc/init/mesos-master.conf`, `/etc/init/mesos-slave.conf` with default configuration files in `/etc/defaults/mesos.conf`. Mesos 0.19 supports external executors, so we use Deimos to run docker containers. This is only required on slaves, but the configuration options are set in the shared `/etc/defaults/mesos.conf` file.

### Marathon and HAProxy

The playbook leverages an `ansible-marathon` role to install a custom build of marathon with Deimos support. If Mesos is the OS for the data center, Marathon is the init system. Marathoin allows us to `http post` new tasks, containing docker container configurations, which will run on Mesos slaves. With HAProxy we can use the masters as a load balancing proxy server routing traffic from known hosts (the masters) to whatever node/port is running the marathon task. HAProxy is configured via a cron job running [a custom bash script][20]. The script queries the marathon API and will route to the appropriate backend by matching a host header prefix to the marathon job name.

### Mesos Followers (Slaves)

The slaves are pretty straightforward. We don&#8217;t need any host variables, so we just list whatever slave nodes you&#8217;d like to configure:

<pre class="brush: plain; title: ; notranslate" title="">[mesos_slaves]
ec2-54-91-78-105.compute-1.amazonaws.com
ec2-54-82-227-223.compute-1.amazonaws.com 
</pre>

Mesos-Slave will be configured with Deimos support.

### The Result

With all this set up you can set up a wildcard domain name, say `*.example.com`, to point to all of your master node ip addresses. If you launch a task like &#8220;www&#8221; you can visit www.example.com and you&#8217;ll hit whatever server is running your application. Let&#8217;s try launching a simple web server which returns the docker container&#8217;s hostname:

<pre class="syntax bash">POST to one of our masters:

POST /v2/apps

{
  "container": {
    "image": "docker:///mhamrah/mesos-sample"
  },
  "cpus": ".25",
  "id": "www",
  "instances": 4,
  "mem": 512,
  "ports": [0],
  "uris": []
}
</pre>

We run four instances allocating 25% of a cpu with an application name of `www`. If we hit `www.example.com`, we&#8217;ll get the hostname of the docker container running on whatever slave node is hosting the task. Deimos will inspect whatever ports are `EXPOSE`d in the docker container and assign a port for Mesos to use. Even though the config script only works on port 80 you can easily reconfigure for your own needs.

To view marathon tasks, simply go to one of your master hosts on port 8080. Marathon will proxy to the correct master. To view mesos tasks, navigate to port 5050 and you&#8217;ll be redirected to the appropriate master. You can also inspect the STDOUT and STDERR of Mesos tasks.

### Notes

In my testing I noticed, on rare occasion, the cluster didn&#8217;t have a leader or marathon wasn&#8217;t running. You can simply restart zookeeper, mesos, or marathon via ansible:

<pre class="syntax bash">#Restart Zookeeper
ansible mesos_masters -a "sudo service zookeeper restart"
</pre>

There&#8217;s a high probability something won&#8217;t work. Check the logs, it took me a while to get things working: grepping `/var/log/syslog` will help, along with `/var/log/upstart/mesos-master.conf`, `mesos-slave.conf` and `marathon.conf`, along with the `/var/log/mesos/`.

### What&#8217;s Next

Cluster schedulers are an exciting tool for running production applications. It&#8217;s never been easier to build, package and deploy services on public, private clouds or bare metal servers. Mesos, with Marathon, offers a cool combination for running docker containers&#8211;and other mesos-based services&#8211;in production. [This Twitter U video highlights how OpenTable uses Mesos for production][21]. The HAProxy approach, albeit simple, offers a way to route traffic to the correct container. HAProxy will detect failures and reroute traffic accordingly.

I didn&#8217;t cover inter-container communication (say, a website requiring a database) but you can use your service-discovery tool of choice to solve the problem. The Mesos-Master nodes provide good &#8220;anchor points&#8221; for known locations to look up stuff; you can always query the marathon api for service discovery. Ansible provides a way to automate the install and configuration of mesos-related tools across multiple nodes so you can have a serious mesos-based platform for testing or production use.

 [1]: http://eurosys2013.tudos.org/wp-content/uploads/2013/paper/Schwarzkopf.pdf
 [2]: https://github.com/GoogleCloudPlatform/kubernetes
 [3]: flynn.io
 [4]: https://github.com/coreos/fleet
 [5]: http://mesos.apache.org/
 [6]: http://www.ansible.com/home
 [7]: http://mesosphere.io/
 [8]: http://spark.apache.org/
 [9]: https://github.com/mesosphere/storm-mesos
 [10]: https://github.com/mesos/hadoop
 [11]: https://github.com/jenkinsci/mesos-plugin
 [12]: https://github.com/airbnb/chronos
 [13]: https://github.com/mesosphere/elasticsearch-mesos
 [14]: https://github.com/mesosphere/cassandra-mesos
 [15]: https://github.com/mesosphere/marathon
 [16]: https://github.com/mesosphere/deimos
 [17]: https://github.com/mhamrah/ansible-mesos-playbook
 [18]: http://mesosphere.io/learn/run-docker-on-mesosphere/
 [19]: https://github.com/mhamrah/ansible-mesos
 [20]: https://github.com/mhamrah/ansible-marathon/blob/master/files/haproxy_dns_cfg
 [21]: https://engineering.twitter.com/university/videos/docker-mesos
