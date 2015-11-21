---
title: How eAccelerator Improved WordPress on My Fedora Server
author: Michael
type: post
date: 2010-02-01
url: /2010/02/how-eaccelerator-improved-wordpress-on-my-fedora-server/
dsq_thread_id:
  - 529628222
categories:
  - Programming
---
I recently moved this blog and some other smaller websites to a virtual machine running on [Rackspace Cloud][1]. So far I&#8217;m loving having my own server, and have been able to get my hands dirty with Linux administration, apache, and mysql.

But one quirk was really bothering me: at times, my WordPress blog would hang. I don&#8217;t get too much traffic, and using

<pre class="brush: bash; title: ; notranslate" title="">top</pre>

showed no real load on the cpu. But there were a lot of Apache threads with a good chunk of memory allocated, and I had a little free memory available.  I tried adding more memory to the server, but no luck.  I thought the issue could be mysql, but running queries wasn&#8217;t a problem.  Neither was static html- the root index.html loaded quickly.  So that left php itself.

A couple of apache config changes didn&#8217;t help.  The thing which really did the trick was installing [eAccelerator][2].  This tool simply keeps the compiled php script available, so apache doesn&#8217;t have to recompile the php script on every load.  No, the blog is a lot faster and much more reliable.

Installation is easy on Fedora (or CentOS, or whatever distro uses yum):

<pre class="brush: bash; title: ; notranslate" title="">sudo yum install php-accelerator</pre>

then just restart apache and you&#8217;re good to go:

<pre class="brush: bash; title: ; notranslate" title="">sudo /sbin/service httpd reload</pre>

If you don&#8217;t notice an immediate improvement, and want to check to make sure it&#8217;s loaded, then create a info.php file with the following code:

<pre class="brush: php; title: ; notranslate" title="">&lt;?php phpinfo(); ?&gt;</pre>

and you should see the accelerator info in the list.

 [1]: http://www.michaelhamrah.com/blog/2010/01/rocking-the-rackspace-cloud/
 [2]: http://eaccelerator.net/
