---
title: Thoughts on Kanban
author: Michael
type: post
date: 2011-12-15
url: /2011/12/thoughts-on-kanban/
dsq_thread_id:
  - 527224506
categories:
  - Programming
tags:
  - Agile
  - development
  - kanban
  - Scrum
---
One of my favorite achievements in the agile/lean world has been the progression from standard Scrum practices to a [Kanban][1] approach of software development. In fact, Kanban, in my opinion, is such an ideal approach to software development I cannot imagine approaching team-based development any other way.

## What&#8217;s Wrong With Scrum?

Before answering this, I want to mention Kanban came only after altering, tweaking, and refining the Scrum process as much as possible. If anything, Kanban represents a _graduation_ from Scrum. Scrum worked, and worked well, but it was time to take the approach to the next level. Why? The Scrum process was failing. It became too constrained, too limiting. As I mentioned in my [three-year old (as of this writing) post on Scrum][2] one needs to constantly iterate in refining the practice. Pick one thing that isn&#8217;t working, fix it, and move one. Quite simply, there was nothing with Scrum left to refine except Scrum itself.

### Why Scrum Was Failing

The main issue was simply it was time to break free from the time-boxed approach of sprints. Too much effort went into putting stories into iterations. Too much effort went into managing the process. This process took away from releasing new functionality. _Nothing can be more important than releasing new functionality._ Tweaking iteration length did not help; one week caused too many meetings to happen too frequently. Two weeks and the early sprint planning effort was lost on stories which would not occur until the second week. Too much time went into making stories &#8220;the right size&#8221;. Some where too small; not worth discussing in a group. Some were too big but they did not make sense to break down to fit into the iteration. Worse, valuable contributions in meetings only occurred with a few people. This had nothing to do with the quality of dev talent; some really good developers did not jive with the story time/sprint review/retrospective/group think model. Why would they? Who really likes meetings?

### Rethinking Constraints

Scrum has a specific approach to constraints: limit by time. Focus on what can be accomplished in X timeframe (sprints). Add those sprints into releases. Wash, rinse, repeat. Kanban, however, rethinks constraints. Time is irrelevant; the constraint is how much work can occur at any one time. This is, essentially, your work in progress. Limit your work in work in progress (WIP) to work you can be actively doing at any one time. In order to do new work, old work must be done.

### Always Be Releasing

The beauty of this approach is that it lends itself well to a continuous deployment approach. If you work on something, and work on it until it is done, when it is done, it can be released. So release it. Why wait until an arbitrary date? The development pipeline in Kanban is similar to Scrum. Stories are prioritized, they are sized, they are ready for work, they are developed, they are tested, they are released. The main difference is instead of doing these at set times, they are done _just-in-time_. In order to move from one stage of the process (analysis, development, testing, etc) there must be an open &#8220;slot&#8221; in the next stage. This is your WIP limit. If there isn&#8217;t an open slot, it cannot move, and stays as is. People can be focused on moving stories through the pipeline rather than meeting arbitrary deadlines, no matter how those deadlines came to be. Even blocking items can have WIP limits. The idea is simple: you have X resources. Map those resources directly to work items as soon as they are available, and see them through to the end. Then start again.

### Everything is Just In Time

All of the benefits of Scrum are apparent in Kanban. Transparency into what is being worked on and the state of stories. Velocity can still be measured; stories are sized and can be timed through the pipeline. Averages can be calculated over time for approximate release dates. The business can prioritize what is next up to the point of development. Bugs can be weaved into the pipeline as necessary, without having to detract from sprints. With the right build and deploy setup releases can occur as soon as code is merged into the master branch. Standup meetings are still important.

### The Goal

The theory of constraints is nothing new. My first encounter was with [The Goal by Eliyahu Goldratt][3]. The goal, in this case, is to release new functionality as efficiently (not quickly, not regularly; efficiently) as possible. There is a process to this: an idea happens, a request comes in. It is evaluated, it is fleshed out, given a cost. It is planned, implemented, and tested. It is released. Some are small, some are big. Some can be broken down. But in teams large and small, they go from inception to implementation to release. Value must be delivered efficiently. It can happen quickly, but it does not need to be arbitrarily time-boxed.

Scrum is a great and effective approach to software development. It helps focus the business and dev teams on thinking about what is next. It is a great way to get teams on board with a goal and working, in sync, together. It follows a predictable pattern to what will happen when. It offers the constraint of time. Kanban offers the constraint of capacity. For software development this is a far more effective constraint to managing work. You still need solid, manageable stories. You just don&#8217;t have to fit a square peg in a round hole. Kanban streamlines the development process so resources, which always have a fixed limit, are the real limit you are dealing with. They are matched directly to the current state of work so a continuous stream of value can be delivered without the stop-and-go Scrum approach.

 [1]: http://en.wikipedia.org/wiki/Kanban_(development)
 [2]: http://www.michaelhamrah.com/blog/2008/12/adventures-in-agile-practical-scrum-intro/
 [3]: http://www.amazon.com/Goal-Process-Ongoing-Improvement/dp/0884270610
