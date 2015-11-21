---
title: Fleet Unit Files for Kubernetes on CoreOS
author: Michael
type: post
date: 2015-06-27
url: /2015/06/fleet-unit-files-for-kubernetes-on-coreos/
dsq_thread_id:
  - 3883050499
categories:
  - Programming
tags:
  - coreos
  - docker
  - kubernetes
---
As I&#8217;ve been leveraging CoreOS more and more for running docker containers, the limitations of Fleet have become apparent.
  
Despite the benefits of [dynamic unit files via the Fleet API][1] there is still a need for fine-grained scheduling, discovery, and more complex dependencies across containers. Thus I&#8217;ve been exploring Kubernetes. Although young, it shows promise for practical usage. The [Kubernetes repository][2] has a plethora of &#8220;getting started&#8221; examples across a variety of environments. There are a few CoreOS related already, but they embed the kubernetes units in a cloud-config file, which may not be what you want.

My preference is to separate the CoreOS cluster setup from the Kubernetes installation. Keeping your CoreOS cloud-config minimal has many benefits, especially on AWS where you can&#8217;t easily update your cloud-config. Plus, you may already have a CoreOS cluster and you just want to deploy Kubernetes on top of it.

The [Fleet Unit Files for Kubernetes on CoreOS are on GitHub][3]. The unit files makes a few assumptions, mainly you are running with a [production setup using central services][4]. For AWS [you can use Cloudformation to manage multiple sets of CoreOS roles as distinct stacks, and join them together via input parameters][5].

 [1]: http://blog.michaelhamrah.com/2015/04/easy-scaling-with-fleet-and-coreos/
 [2]: https://github.com/GoogleCloudPlatform/kubernetes
 [3]: https://github.com/mhamrah/kubernetes-coreos-units
 [4]: https://coreos.com/docs/cluster-management/setup/cluster-architectures/#production-cluster-with-central-services
 [5]: http://blog.michaelhamrah.com/2015/03/managing-coreos-clusters-on-aws-with-cloudformation/
