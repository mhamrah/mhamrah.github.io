---
title: Rocking the Rackspace Cloud
author: Michael
type: post
date: 2010-01-13
url: /2010/01/rocking-the-rackspace-cloud/
dsq_thread_id:
  - 527224554
categories:
  - Programming
---
I&#8217;ve been unhappy with my current hosting provider and in looking for an alternative I tried [Rackspace Cloud Server][1].  I&#8217;ve wanted a dedicated server for a while, but it&#8217;s been cost prohibitive compared to shared machine hosting.  This blog, as well as some other, smaller sites, are now running on a Rackspace Cloud Server.  Needless to say I love it.  Rackspace Cloud Server is simply awesome.

**Setup**

Signing up and setup was fast and simple.  There was a weird account verification process where Rackspace had to call me to finalize and confirm setup, but I only had to wait about ten minutes after I signed up for the call.  After that, it was only a couple of clicks before I was logging in to my new server.  There are several Linux images to choose from- you select the size of the machine you want and the os- and within seconds you have a public ip v4 address and root access. For me that was the most amazing part of cloud computing- I wanted a new server, and I got one with instant gratification.

**Getting Started**

I&#8217;m not a Linux expert coming from the Windows world, but I was able to configure everything and install mysql/apache in no time.  Rackspace has a [series of extremely well written and straightforward how-to&#8217;s to get your server configured correctly][2].  Each OS image is a stripped down bare-bones install so there&#8217;s some setup required, but by using the knowledge base I was up and running quickly.  Backups are integrated into the management website with Rackspace&#8217;s Cloud Files product, so I took a snapshot for reuse later.  You can even schedule snapshots as needed.

It&#8217;s really the knowledge base which I loved- I got ftp, apache, php, ssh, mysql all installed and secured quickly.  There&#8217;s a wealth of information there.  If you need to learn Linux or do anything with popular OS applications check out their kb. I&#8217;ve never seen so much easy to access documentation on open source software in one place.

**Other Goodies
  
** 

The reason why I wanted to try Rackspace is because the cost of entry is extremely cheap- it starts at 1.5 cents/hr which comes out to about $11/month.  Much cheaper than Amazon&#8217;s EC2, which starts around $20 only if you prepay.  There are a lot of major differences between the two products which I talk about below.

Rackspace Servers come in different sizes all differentiated by the amount of RAM.  I chose the 256mb option because it was the cheapest.  It turns out it was a little too light on power for what I wanted, so I upgraded to the 512mb option.  The upgrade was seemless- you just say &#8220;make this a 512mb server&#8221;, and Rackspace queues the request, reboots when appropriate, you verify, and then you get the same server with better &#8220;hardware&#8221;.  Loved it.  I queued the request before I left work and it was good to go when I got home.

The best part about Rackspace is the support.  There&#8217;s a live chat link from the management page.  I had a question about DNS- I clicked live chat- had a quick IM conversation- got the answer- and I was done.  That&#8217;s service.

**Rackspace vs. EC2**

The main reason I chose Rackspace over EC2 was cost.  But in looking into the differences between Rackspace and EC2 I&#8217;m happy I chose Rackspace.  The biggest difference is with Rackspace you can reboot the OS without losing state- so if you make configuration change or install some software, a reboot behaves like you&#8217;d expect- everything stays the same.  EC2, on the other hand, resets the server state back to what it was on the first boot.  You need to have your EC2 server link to EBS to persist state. WIth Rackspace you get a hard drive which acts, surprisingly, just like a hard drive.

With Rackspace, the default is ip v4, but with AWS, it&#8217;s ip v6 and you need to link to an ip v4 address.  The initial setup is also slightly different- EC2 involves the use of a keypair for the initial login, but Rackspace gives you root access which you then change and configure as needed.  I haven&#8217;t actually set up an EC2 server but by looking at the how to I an safely say Rackspace is a lot simpler.

Another major difference in options is the &#8220;size&#8221; of the server.  EC2 is explicit in the hardware configuration- RAM, cpu power, cores, etc.  Size is capped.  Rackspace takes a different approach: all CPUs are 64 bit with four cores and you only choose the amount of RAM.  Storage and CPU power change with the size of RAM.  You&#8217;re guaranteed a certain minimum power (rather than a capped maximum), and if more power is available you get it.

I can say EC2 has much more options available for OSes- EC2 offers Windows in addition to Linux (apparently this is coming soon to Rackspace).  I also like the idea of EC2&#8217;s community AMI&#8217;s.  You only get a bare bones OS with Rackspace, but with EC2 you can get a fully configured LAMP stack or some other ready-to-go server.  But it was fun getting everything set up myself with Rackspace.  I haven&#8217;t done a long haul with either product, and like I said I haven&#8217;t actually spun up an EC2 instance, so I can&#8217;t get into any more descriptive details.

**Rocking the Cloud**

I can&#8217;t tell you how fun it is to be able to spin up a new server out of the blue.  There are a million things I want to do- being able to run a server full time, with great bandwidth, without having to find a place for it in my house (or buy and salvage hardware) is just geeky awesome.  And for only $11/mo!  It&#8217;s crazy!  Whether with Rackspace or Amazon I encourage you to give it a shot.  I encode a lot of video for my ipod, which means running my laptop at night or slow performance when doing other things.  I&#8217;d love to be able to spin up a server, upload a video, encode it, pull it down, and shut down the server to save money.  Power on demand!

Disclaimer: I own stock in both Rackspace and Amazon, but I wasn&#8217;t paid nor asked to write this article.

 [1]: http://www.rackspacecloud.com/cloud_hosting_products/servers
 [2]: http://cloudservers.rackspacecloud.com/index.php/Main_Page
