---
title: 'Choosing a Technology: You’re Asking the Wrong Question'
author: Michael
type: post
date: 2013-03-27
url: /2013/03/choosing-a-technology-youre-asking-the-wrong-question/
dsq_thread_id:
  - 1169033640
categories:
  - Programming
---
When making a choice in the tech world there are two wide-spread approaches: &#8220;What&#8217;s better, X or Y?&#8221; and &#8220;Should I use xyz?&#8221;. The &#8220;or&#8221; debate is always an entertaining topic usually ending in an absurdly hilarious flame war. The &#8220;Should I use xyz?&#8221; is a subtler, more prevalent question in the tech community leading to an extensive amount of discourse. Fairly rational, usually with some good insight, but still a time consuming task. I&#8217;ve fallen victim to both approaches when exploring a technology decision. What I realized is I&#8217;m asking the wrong question. There are only two things I should ask:

1) What problem do I need to a solve?
  
2) How do I want to solve it?

Once I take this approach I have an opinionated basis for decision-making and I have a clear direction in how to make that decision. Frameworks&#8211;web or javascript&#8211;are excellent examples on taking this approach. Most of these frameworks were born on the simple premise of solving a problem in an opinionated way. Backbone takes a bare-bones approach to a front-end, event-driven structure; Ember offers a robust, &#8220;things just happen&#8221; framework. Sinatra and co. offers an http-first approach to development. Rails and variants are opinionated in web application structure. Do you agree with that approach? Yes, excellent! No? Find something else or roll your own.

Don&#8217;t know the answer? That&#8217;s okay too. Most beginners want to make the &#8220;right&#8221; choice on what to learn. But the thing is there is no &#8220;right&#8221; answer. For a beginner choosing python vs. ruby vs. php vs scala wastes effort. Just build something using something: you&#8217;ll soon develop your own opinions, with &#8220;how easy is this to learn&#8221; probably the first. Next, when your rails codebase is out of control and you&#8217;re drowning in method_missing issues maybe you&#8217;ll want a more granular, service-orientated approach and the type-safety of Scala. Maybe not&#8230; But you&#8217;ll have a valid problem to solve and a reasonable opinion to go with it.

I suggest reading [Andrew Alexeev&#8217;s reason on why he built NGINX][1] and [Poul-Henning Kamp&#8217;s rationale on how you write a modern application][2]. Like so many others these incredible open-source systems were born from a problem and the way someone wanted it solved. But those systems didn&#8217;t happen overnight and the authors didn&#8217;t start from scratch. They spent years encountering, learning, and dealing with problems in their respective spaces. They knew the problem domain well, they knew how they wanted the problem solved, and they solved it.

So put your choice in a context and don&#8217;t sweat the details which are irrelevant to the task at hand. When you need to know those details you&#8217;ll know them, and when you hit problems you&#8217;ll know how you want them solved.

 [1]: http://www.aosabook.org/en/nginx.html
 [2]: https://www.varnish-cache.org/trac/wiki/ArchitectNotes
