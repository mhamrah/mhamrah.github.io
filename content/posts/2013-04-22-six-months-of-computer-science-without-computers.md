---
title: Six Months of Computer Science Without Computers
author: Michael
type: post
date: 2013-04-22
url: /2013/04/six-months-of-computer-science-without-computers/
dsq_thread_id:
  - 1227185293
categories:
  - Programming
---
A few weeks ago I returned from [a six month trip around Asia][1]. I didn&#8217;t have a computer while abroad, but I was able to catch up on several tech books I never had time for previously. Reading about programming without actually programming was an interesting and rewarding circumstance. It provided a unique mental model: it was no longer about &#8220;how you do this&#8221; but about &#8220;why would you do this&#8221;. Accomplishment of a task via implementation was not an end goal. The end goal was simply absorbing information; once read, it didn&#8217;t need to be applied. It only needed to be reasoned about and hypothetically applied under a specific situation (which I usually did on a trek or on a beach). Before I would have been eager to try it out, hacking away, but without a computer, I couldn&#8217;t. It was liberating. Given a problem, and a set of constraints, what&#8217;s the ideal solution? I realize this is somewhat of an ivory-tower mentality, however, I also realized some of the best software has emerged from an idealism to solve problems in an opinionated way. Sometimes we are too consumed by the here-and-now we fail to step back for the bigger picture. Conversely, we hold onto our ideals and fail to adapt to changing circumstances.

My favorite aspect of learning technology while traveling abroad did not come from any book or video. A large part of computer science is about optimizing systems under the pressure of constraints. Efficient algorithms, clean code, improving performance. The world is full of sub-optimal processes. Burmese hotels, the Lao transportation system, and Nepalese immigration to name a few. On a larger scale sun-optimal problems are created by geographic, socio-economic, or political constraints. People try the best they can to improve their way of life, and unfortunately, the processes are often &#8220;implemented&#8221; with a &#8220;naïve&#8221; solution. Some are also inspiring. It was powerful to see these systems up close, with cultural and historical factors so foreign. One thing is certain: when you optimize for efficiency, everyone wins.

Below are a selection of books and resources I found particularly interesting. I encourage you to check them out, hopefully away from a computer in a foreign land:

[97 Things Every Programmer Should Know][2] : A great selection of tidbits from a variety of sources. Nothing new for the experienced programmer, but reading through the sections is a great refresher to keep core principles fresh. Worthwhile to randomly select a chapter now and again for those &#8220;oh yeah&#8221; moments.

[Beautiful Code][3] by Andy Oram and Greg Wilson: My favorite book. Not so much about code, but the insight about solving problems makes it a great read. I appreciate the intelligent thought process which went into some of the chapters. Python&#8217;s hashtable implementation and debugging prioritization in the Linux kernel are two highlights.

[Exploring Everyday Things with R and Ruby][4] by Sau Sheong Chang: This is a short book with great content. You only need an elementary knowledge of programming and mathematics to appreciate the concepts. It&#8217;s also a great way to get a taste of R. The book covers a variety of topics from statistics, machine learning, and simulations. My favorite aspect is how to use modeling to verify a hypothesis or create a simulation. The chapters involving emergent behavior are particularly interesting.

[Machine Learning for Hackers][5] by Drew Conway and John Myles White: I&#8217;ve been interested in machine learning for a while, and I was very happy with this read. Far more technical and mathematical than _Exploring Everyday Things_, this book digs into supervised and unsupervised learning and several aspects of statistics. If you&#8217;re interested in data science and are comfortable with programming, this book is for you.

[Scala in Action][6] by Nilanjan Raychaudhuri: Scala and Go have been on my radar for a while as new languages to learn. It&#8217;s funny to learn a new programming language without being able to test-drive it, but I appreciated the separation. My career has largely been focused on OOP: leveraging design patterns, class composition, SOLID principles, enterprise architecture. After reading this book I realize I was missing out on great functional programming paradigms I was only unconsciously using. Languages like Clojure and Haskell are gaining steam for a radically different approach to OOP, and Scala provides a nice balance between the two. It&#8217;s also wonderfully expressive: traits, the type system, and for-comprehension are beautiful building blocks to managing complex behavior. Since returning I&#8217;ve been doing Scala full-time and couldn&#8217;t be happier. It&#8217;s everything you need with a statically typed language with everything you want from a dynamic one (well, there&#8217;s still no method-missing, at least not yet). I looked at a few Scala books and this is easily on the top of the list. Nilanjan does an excellent job balancing language fundamentals with applied patterns.

[HBase: The Definitive Guide][7] by Lars George: I&#8217;ve been deeply interested in distributed databases and performance for some time. I purchased this book a few years ago when first exploring NoSQL databases. Since then, Cassandra has eclipsed the distributed hashtable family of databases (Riak, Hbase, Voldemort) but I found this book a great read. No matter what implementation you go with, this book will help you think in a column-orientated way, offering great tidbits into architectural tradeoffs which went into HBase&#8217;s design. At the very least, this book will give you a solid foundation to compare against other BigTable/Dynamo clones.

[The Architecture of Open Source Applications][8]: I was excited when I stumbled upon this website. It offers a plethora of information from elite contributors. The applied-practices and deep architectural insight are valuable lessons to learn from. [Andrew Alexeev on Nginx][9], [Kate Matsudaira on Scalable Web Architecture][10] and [Martin Sústrik on ZeroMQ][11] are highlights.

## iTunes U

I was also able to check out some courses on iTunes U while traveling. [The MIT OCW Performance Engineering of Software Systems][12] was my favorite. Prof. Saman Amarasinghe and Prof. Charles Leiserson were both entertaining lecturers, and the course provided great insight into memory management, parallel programming, hardware architecture, and bit hacking. I also watched several lectures on algorithms giving me a new found appreciation for Big-O notation (I wish I remembered more while on the job interview circuit). I&#8217;ve been gradually neglecting the importance of algorithmic design since graduating ten years ago, but found revisiting sorting algorithms, dynamic programming, and graph algorithms refreshing. Focusing on how well code runs is as important as how well it&#8217;s written. Like most things, there&#8217;s a naïve brute-force solution and an elegant, efficient other solution. You may not know what the other solution is, but knowing there&#8217;s one lurking behind the curtain will make you a better engineer.

So, if you can (and you definitely can!) take a break, grab a book, read it distraction free, gaze out in space and think. You&#8217;ll like what you&#8217;ll find!

 [1]: http://thegreatbigadventure.tumblr.com
 [2]: http://programmer.97things.oreilly.com/wiki/index.php/97_Things_Every_Programmer_Should_Know
 [3]: http://shop.oreilly.com/product/9780596510046.do
 [4]: http://shop.oreilly.com/product/0636920022626.do
 [5]: http://shop.oreilly.com/product/0636920018483.do
 [6]: http://www.manning.com/raychaudhuri/
 [7]: http://shop.oreilly.com/product/0636920014348.do
 [8]: http://www.aosabook.org/en/index.html
 [9]: http://www.aosabook.org/en/nginx.html
 [10]: http://www.aosabook.org/en/distsys.html
 [11]: http://www.aosabook.org/en/zeromq.html
 [12]: http://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-172-performance-engineering-of-software-systems-fall-2010/index.htm
