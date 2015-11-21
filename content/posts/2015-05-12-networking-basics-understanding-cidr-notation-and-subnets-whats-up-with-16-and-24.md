---
title: 'Networking Basics: Understanding CIDR notation and Subnets: what’s up with /16 and /24?'
author: Michael
type: post
date: 2015-05-12
url: /2015/05/networking-basics-understanding-cidr-notation-and-subnets-whats-up-with-16-and-24/
dsq_thread_id:
  - 3758564410
categories:
  - Programming
tags:
  - cidr
  - networking
  - subnet masks
---
An IP address (specifically, an IPv4 address), like 192.168.1.51, is really just 32 bits of data. 32 ordinary bits, like a 32 bit integer, but represented a little differently than a normal int32. These 32 bits are split up into 4 8-bit blocks, with each block represented as a number with a dot in between. This is called dotted-decimal notation: 192.168.1.51. Curious why an IP address never has a number in it greater than 255? That&#8217;s the maximum value for 8 bits of data: you can only represent 0 to 255 with 8 bits.

The IP address 192.168.1.51 in binary is 11000000 10101000 00000001 00110011, or just 11000000101010000000000100110011. If this were an int32 it would be 3232235827. Why not just use this number as an address? By breaking up the address into blocks we can logically group related ip addresses together. This is helpful for routing and subnets (as in sub-networks) which is where we usually see [CIDR notation][1]. It lets us specify a common range of IP address by saying some bits in the IP address are fixed, while others can change. When we add a slash and a number between 0 and 32 after an IP, like 192.168.1.1/24, we specify which bits of the address are fixed and which can be changed.

Let&#8217;s take a look at Docker networking. Docker creates a subnet for all containers on a machine. It uses CIDR notation to specify a range of IP address for containers, usually 172.17.42.1/16. The /16 tells us the first sixteen bits in the dotted-decimal ip address are fixed. So any ip address, in binary, must begin with 1010110000010001. We are &#8220;masking&#8221; the first 16 bits of the address. Docker could assign a container of 172.23.42.1 but could not assign it 172.32.42.1. That&#8217;s because the first four bits in 23 are the same as 17 (0001) but the first four bits 32 are different (0010).

Another way to specify a range of IP addresses is with a &#8220;subnet mask&#8221;. Instead of simply saying 172.17.42.1/16, we could give the ip address of 172.17.42.1 and a subnet mask of 255.255.0.0. We often see this with tools like `ifconfig`.

255.255.0.0 in binary is 11111111 11111111 00000000 00000000. The first sixteen bits are 1 telling us which bits in the corresponding IP address are fixed. Sometimes you see a subnet mask of 255.240.0.0: that&#8217;s the same as an ip range of /12. The number 240 is 11110000 in binary, masking the first 12 bits in the subnet mask 255.240.0.0 (8 1&#8217;s for 255, and 4 1&#8217;s for 240). You could never have a subnet of 239, because there is a 0 in the middle of the binary number 239 (11101111), defeating the purpose of a mask.

A /16 range gives us 2 8 bit blocks to play with, or 65535 combinations: 2^16, or 256\*256. But the 0 and 255 numbers are reserved in the IP address space, so we really only get 254 \* 254 combinations, or 64516 addresses. This is important when setting up networks: if you want to make sure you have enough IP addresses for all the things in your network. A software defined network like [Flannel][2], which allows you to create a private subnet for docker containers across hosts, uses a /16 subnet for a cluster and /24 range per node. So an entire cluster can only have 64516 ip address. More specifically, it can only have 254 nodes running 254 containers. Large networks, like those at most companies, use the reserved address space 10.0.0.0/8. The first 8 bits are fixed giving us 24 bits to play with: or 16,387,064 possible addresses (because we can&#8217;t use 0 or 255). These are usually broken up into several subnet works, like 10.252.0.0/16 and 10.12.0.0/16, carving up smaller subnets from the larger address space.

Subnet masks and CIDR notation play prominent roles in a variety of areas beyond specifying subnets. They are heavily used in routing tables to specify where traffic for a particular IP should go. They are used extensively in AWS and other cloud providers to specify firewall rules for security groups as well as their VPC product. Understanding CIDR notion and subnet masks make other aspects of networking: interfaces, gateways, routing tables much easier to understand.

 [1]: http://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
 [2]: https://github.com/coreos/flannel
