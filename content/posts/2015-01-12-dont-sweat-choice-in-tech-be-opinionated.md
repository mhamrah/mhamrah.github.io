---
title: 'Don’t Sweat Choice in Tech: Be Opinionated'
author: Michael
type: post
date: 2015-01-12
url: /2015/01/dont-sweat-choice-in-tech-be-opinionated/
dsq_thread_id:
  - 3413544156
categories:
  - Programming
tags:
  - development
  - technology
---
Recently I jumped back into some front-end development. My focus is primarily on backend systems and APIs so I welcomed the opportunity to hack on a UI. I keep tabs on the front-end world and a new project is a good opportunity for a test-drive or to level-up on an existing toolkit. The caveat, however, is the dreaded [developaralysis][1]. We have so many choices that discerning the difference, picking the &#8220;right one&#8221;, and learning it becomes an overwhelming endeavor. Should I try out [gulp][2]? What about test-driving [react][3]? Or should I go with the usual bootstrap/angular combo I&#8217;ve come to know well?

It&#8217;s hard to balance the cost of time in the present for the potential&#8211;and I emphasize potential&#8211;benefit of speed and simplicity later when choosing something new. Time is limited; do I need an exploration of browserify, amd, and umd when all I really need is a simple `script` tag? Browserify looks cool, but what&#8217;s the return on that investment? The flood of options occurs at every level of experience; it&#8217;s endearing to overhear a debate amongst new developers on whether to learn rails or node first. It&#8217;s definitely not helpful when [sites offer laundry lists of languages you should learn][4]. C# and Java, really? I&#8217;m surprised assembly wasn&#8217;t on the list. Judging the nuances of NoSQL options is just as entertaining.

My programming career, now inching the 15-year mark, has seen its fair share of languages and frameworks. Happily I no longer think about ASP.NET view state or the server-control lifecycle. These were instrumental at one time, and even though they are long gone, those experiences helped shape my current opinions on how I want to develop (or, in this particular case, not to develop) software. I didn&#8217;t realize I had a choice back then on how I develop: ASP.NET seemed a given. That horrible windows-on-web paradigm pushed me to an intense focus on MVC, now a staple of many web frameworks. In turn, with the advent of APIs and more complex, task-focused UX, I am keenly interested in stream-based programming emerging in [Scala][5] and [Node][6]. What I&#8217;ve come to realize in the tumultuous world of programming, and with constantly needing to level-up, is that frameworks and languages are only part of the equation. The most important part is me, and you: the developer. Steve Ballmer got it right: [it&#8217;s about developers][7]. Languages and frameworks help us do things but our opinions on how we want to do them is what moves us forward. When a framework matches your opinions, getting stuff done is simple and intuitive. When you feel like your jumping through hoops it&#8217;s time to try something different.

My small foray back into the front-end world was met with the usual whirlwind of information. Not to mention the usual upgrade of tools, so any answers I find on google will be outdated:

<blockquote class="twitter-tweet" lang="en">
  <p>
    &#8220;What&#8217;s bower?&#8221; &#8220;A package manager, install it with npm.&#8221; &#8220;What&#8217;s npm?&#8221; &#8220;A package manager, you can install it with brew&#8221; &#8220;What&#8217;s brew?&#8221; &#8230;
  </p>
  
  <p>
    — Stefan Baumgartner (@ddprrt) <a href="https://twitter.com/ddprrt/status/529909875347030016">November 5, 2014</a>
  </p>
</blockquote>



I just want to throw together a web site. Grunt vs. Gulp? Wait, there&#8217;s this thing called [Broccoli][8]? Is there something different than [Bootstrap][9] that&#8217;s less bootstrappy? Are people still using [h5bp][10]?

It seemed even the simple go-to of [`yo angular`][11] was fraught with peril: what are all these questions I have to answer? A massive amount of files were generated. Yes, all important and I know all required for various things, but it&#8217;s information overload. Why is `unresolved` added to the body tag? What happens if I hack the meta-viewport settings? What is `build:js` doing? Should I put on the blinders and ignore? Maybe I should try the possible simplicity of [just using npm][12]. I was in it: developaralysis.

Stop. Relax. Breathe. I already knew the simple answer to navigating the awesome amount of choice: opinion. Forget about existing, pre-conceived notions of software. You need to do something: how would you do it? Chances are someone&#8217;s had a similar idea and [wrote some software][13]. Don&#8217;t even know what you need? Then start with something that&#8217;s easy to learn. If it doesn&#8217;t work out, you&#8217;ll have formed an opinion on what you wanted to happen. This is learning by fire.

Opinions have given birth to some of the most widely used software in the world. [Yukihiro Matsumoto created Ruby from his dissatisfaction with other OOP languages][14]. [Nginx was spawned by dissatisfaction in threaded web-servers][15]. You may not be ready to write a new language, framework, or web server, but your opinions can still shape what you learn and where you invest your time.

Nobody asked me a decade ago how I wanted to write web software. If they did I doubt I would have come up with anything similar to ASP.NET webforms, if I could even have put together a semi-coherent answer. Yet I was a full-time ASP.NET webforms developer, and that&#8217;s how I was writing software. Eventually my teammates and I realized this way of programming was utter crap. We asked ourselves that simple question, which we should have asked a lot earlier: How do we want to do this?

At any level it&#8217;s important to develop opinions on how you want to achieve goals. Beginners may seem they have a difficult spot because there&#8217;s so little grounding to formulate opinions: any answer may appear &#8220;too simple&#8221;. But the spectrum is the same for experience developers as well. There&#8217;s always &#8220;something else&#8221; to know and factor in behind the curtain. You&#8217;re constantly peeling layers off of the onion.

Don&#8217;t sweat the plethora of choice which exist. Nothing is perfect, and stagnation is the worst option. Take a moment and develop an opinion on how you want to solve a particular problem. Poke holes in your solution. See if somebody else has a similar idea, or a similar experience. Try something out: don&#8217;t like how it happened or the result? Did you leverage the tool correctly? Okay, great, now you have the basis for something better. Develop your suite of go-to tooling. You can keep tabs on the eco-system, and cross-pollinate ideas across similar veins. Choice is a good thing: like a breadth-first search, letting you still run forward if you want.

 [1]: http://techcrunch.com/2014/10/18/you-too-may-be-a-victim-of-developaralysis/
 [2]: http://gulpjs.com
 [3]: http://facebook.github.io/react/
 [4]: http://mashable.com/2014/01/21/learn-programming-languages/
 [5]: http://doc.akka.io/docs/akka-stream-and-http-experimental/1.0-M2/scala/stream-index.html
 [6]: https://github.com/substack/stream-handbook
 [7]: http://vimeo.com/6668315
 [8]: https://github.com/broccolijs/broccoli
 [9]: http://getbootstrap.com
 [10]: http://html5boilerplate.com
 [11]: http://yeoman.io
 [12]: http://blog.keithcirkel.co.uk/why-we-should-stop-using-grunt/
 [13]: https://github.com/explore
 [14]: http://en.wikipedia.org/wiki/Ruby_(programming_language)#Early_concept
 [15]: http://www.aosabook.org/en/nginx.html
