---
title: 'Scrum Tips: Managing Task Items'
author: Michael
type: post
date: 2009-03-11
url: /2009/03/scrum-tips-managing-task-items/
dsq_thread_id:
  - 527224600
categories:
  - Programming
tags:
  - Agile
  - Scrum
---
One of the biggest hurdles in transitioning to scrum is getting everyone- especially the dev team- on the same page.  Sprint planning and even scrum meetings can be painfully boring and seem useless.  There&#8217;s a lack of understanding of what to do, communication is either too little or too much, and a few people bad mouth the process and continue their heads down development.  You&#8217;re left wondering what&#8217;s the point- thinking it&#8217;s never going to work.  Well, have faith: proper task management is key in making scrum work for you and your team.  Task management is an aspect of scrum that&#8217;s entirely contained within the dev group, which makes implementation a lot easier.

#### Have a Purpose

The key is you- the scrummaster- need to provide an environment (like a framework) for scrum.  If you think by simply sending some scrum info around, meeting every day, and having a backlog will magically make everything better you&#8217;re wrong and you&#8217;ll fail.  You&#8217;ll also make everyone miserable.  The reason why &#8220;It gets worse before it gets better&#8221; is because it takes time to transition the team into the scrum process.  The key to success is facilitating that transition as quickly as possible, and the solution is having a purpose for everything you do.

That purpose comes via two simple rules which will help you and your team transition to scrum faster:

  1. **All developers should work on the same backlog item.**
  2. **No task item should take more than four hours.**

Think of these rules as grease in the wheels of scrum.  It keeps the machine moving.

#### A Quick Refresher

A backlog item and task item are two different things.  A backlog item is something that&#8217;s business focused.  It&#8217;s specifically created by the product owner for the development team to implement.  It&#8217;s not, and shouldn&#8217;t be, developer focused in any way.  A task item, on the other hand, is developer focused.  It&#8217;s a list of things needed to be done to finish a backlog item.  Task items are created in either a sprint planning or a developer meeting at the start of a sprint.  They can also be refactored in the middle of the sprint.

A crucial step in scrum is having the team gauge complexity of backlog items.  This requires developers to have an understanding of what needs to be done from a user perspective- which is not something developers are usually good at.  Without understanding what needs to be done, they don&#8217;t know how to do it, and they can&#8217;t tell you how long it will take.  Another popular excuse is &#8220;I need to start coding to figure this out&#8221;.

The key is to make developers think and plan first so there are fewer variables mid sprint.  This is not mini-waterfall.  It&#8217;s strategy.

#### How the Rules Help

Having everyone work on the same item is essential for scrum.  Why? If people are working on different items there&#8217;s little reason for them to communicate.  If Developer A is working on feature X, and Developer B is working on feature Y, there&#8217;s no reason for A or B to care what the other is doing.  In fact, it&#8217;s a waste of time.  However, if both A and B are working on feature X, there&#8217;s every reason in the world for the two to talk.  Not only talk, but collaborate.  It&#8217;s very XP without the side by side awkwardness.  The major benefit is the entire team is focused on the highest priority item- which means something will get completed in the sprint.  By working on single items together each item gets finished faster.  QA has more time to test- and they do it one at a time rather than dealing with multiple things coming together at the end of the sprint.

#### Break Things Down

The four hour rule is key to facilitate working together.  By breaking down every task into small pieces, more analysis is done on each backlog item.  The trick is each task is a single thing which can be done and checked in-  a class creation, a new method- all with unit tests.  The benefits materialize in different ways.  First, sprint planning becomes much more effective as communication occurs via the process of breaking down items.  Analysis is done and development ideas are thought out and debated amongst team members.  A roadmap is created outlining steps needed to complete each item- everyone is in the know and on the bus.

Why four hours? Easy: in theory, a developer should be able to get two four-hour tasks done between scrums.  A developer should never be in a position to say, &#8220;I&#8217;m working on X, and I&#8217;m still planning on working on X&#8221;.   (Unless, of course, they fall behind- in which case, break down the task again).  The point is you want to focus on granularity.  If you have to do a controller, a task of &#8220;Write a controller&#8221; is not adequate.  What are you doing? What actions are needed? What dependencies are there? Are you using ModelBinders, or method parameters? Where&#8217;s the error handling, if any?  The API and code structure evolves out of granular task items- and the whole team is part of the process, everyone on the same page as to what needs to get done. This level of discussion, collaboration, and task tracking are what you&#8217;re looking for.

Of course, it&#8217;s hard to get the entire team working on backlog items when they&#8217;re small.  It&#8217;s really your judgment call- but in my experience, there are only a few tasks which are too small to be worked on by more than one person in less than four hours.  The end of completing a backlog item is the most difficult when it&#8217;s a lot of little to dos.  But the things which take longer then they should are always the items worked on by one person- which usually aren&#8217;t properly tasked out.  The bottom line is there&#8217;s a constant cycle of breaking things down and redistribution.

**TDD**

A key factor to success is TDD.  TDD allows isolation of the moving parts.  In this way, a developer can work on a class, a method, or a service without having to worry about dependent objects.  The task description and sprint planning identify what the moving parts are, and how the moving parts fit together.  TDD is the environment to implement those moving parts, and make sure expectations are being met.  You&#8217;ll usually end up rewriting task items mid sprint.  That&#8217;s okay.  Sprint planing is really about establishing guidelines for development.  TDD is where the code and project materializes, and you need to change course when necessary as the project evolves.

Unit Tests provide a great way to review code.  If you&#8217;re working on a data layer for a business object, the business object developer can look at the data layer tests and get all the usage available.  Little time is spent &#8220;figuring it out&#8221;.  And if the business object developers mock objects don&#8217;t match the concrete ones, somebody wasn&#8217;t on the same page.  However, with small, iterative tasks taking a step back and refactoring is a non-issue.

#### A Better Perspective from Above

A great result of this process is the development lead or architect has great insight to how the application shapes up.  During sprint plannings, guidelines can be set on how to implement backlog items.  If something isn&#8217;t tasked out correctly, point out the issue and discuss.  Dependencies can be identified and spun off into new classes.  When issues arise during scrums, refactoring can happen and new directions can be taken- all in the presence of senior devs, which minimize one offs or crazy spaghetti code.  BTW writing spaghetti code is nearly impossible- it&#8217;s hard to make a mess in less than four hours when you&#8217;re only doing one specific thing and everyone is watching.

#### Mentoring

This process works great for junior developers- they can learn from the entire team.  Similar to an architect having insight to a system, a junior developer has better access to seasoned developers.  One on one time is reduced in favor sprint planning and scrums.  Their work can be evaluated when code comes together for a backlog item.  Any developer is capable of answering a question because everyone is working on the same thing.

#### Cleaner Code

Code reviews happen naturally when code is shared for feature development.  Consider the usual application stack: UI, Controller, Application, Domain, Data Layer.   This stack is no longer in the hands of one person, who either blurs the line between layers or creates a million helper classes.  Code review happens when the pieces gets fused together.  If the puzzle pieces don&#8217;t fit, refactor.  They usually will fit because everyone is clear on task items- because they were small, granular, and created together.

No one person is left to the whim of their own devices- the expectation is clear and apparent to the team during scrums.  And the best developers have a better surface area for showing off, by being exposed to the entire application and not regulated to a single feature. You&#8217;ll also get better exposure to cross functional teams, like UI.  When the Javascript developer and the backend developer are in the same room, creating the same roadmap, knowing what needs to be done, development is truly fluid.

#### Conclusion

Scrum, at its core, is really about structure.  Each part is an important gear on the clock which helps the hands move smoothly.  A key to getting the team working in that structure is setting up an environment that meets the .  Task management is key for managing developers and the sprint.  The four hour task breakdown and single item focus are two tools for managing complexity and fostering communication.  The end result are the following benefits:

  1. Better gauge complexity of backlog items.
  2. Complete work in priority order.
  3. Fully meet Got Done criteria at end of sprint.
  4. Understand where things are with development mid-sprint.
  5. Understand why things are delayed with development mid-sprint.
  6. Fix cohesion with development.  Specifically, enforce development standards and architecture guidelines, write clean code, and minimize hacks and bugs.
  7. Top down visibility for architects, bottom up visibility for developers.

Remember, task management is only part of the entire scrum process.  In order for scrum to be effective, everything must fall into place.  But task management will have an immediate benefit for you and your development team.

[<img src="http://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fwww.michaelhamrah.com%2fblog%2findex.php%2f2009%2f03%2fscrum-tips-managing-task-items%2f&bgcolor=000099" border="0" alt="kick it on DotNetKicks.com" />][1]

 [1]: http://www.dotnetkicks.com/kick/?url=http%3a%2f%2fwww.michaelhamrah.com%2fblog%2findex.php%2f2009%2f03%2fscrum-tips-managing-task-items%2f
