---
title: 'Hiring Software Developers: The In House Interview (Open Ended Questions)'
author: Michael
type: post
date: 2008-12-10
url: /2008/12/hiring-software-developers-the-in-house-interview-open-ended-questions/
dsq_thread_id:
  - 527224428
categories:
  - Programming
tags:
  - hiring
  - interview
---
_Note: This is part of a series on hiring software developers.  See <a title="Interview" href="http://www.michaelhamrah.com/blog/index.php/tag/interview/" target="_self">articles tagged with interview for the series</a>.  I needed to break the in house interview post into two parts.  This part focuses on open ended questions._

The in house interview is by far the most important part of the hiring process.  It&#8217;s also the most difficult.  You need to vet a person thoroughly in a very short time.  If you don&#8217;t have a strategy for the in house interview you will not effectively gauge a candidate nor be able to make a confident &#8216;yes&#8217; decision.  Furthermore, if you enter into a bargaining/negotiating process, you want enough information to know how far to go in the negotiations.

**Open Ended Questions**

Quite simply, you&#8217;re testing a developer on how well they solve problems.  No, not those ridiculous how many dentists are in San Fransisco questions.  Development problems- that measure a candidate in how well they analyze information, gather requirements, structure applications, organize data, implement software and scale for the future.  These are development- not programming- problems.  They&#8217;re not algorithms or set theory or knowing the best collection class to use.  These are problems regarding how well you go from a customer or product owner telling you something to a living, breathing software system.  If you&#8217;re an agile development shop, or want to be one, it&#8217;s essential you hire developers- not programmers- who do this well.

Open ended questions offer a suite of scenarios where you explore various skills that define a good developer.  These scenarios are really up to you and can include everything from data modeling to application architecture to user interface design.  Essentially, you simply tee up a situation and see how the developer deals with the situation.  There&#8217;s no right or wrong answer, just good or bad discussions.  More importantly, it requires solid communication- the ability to understand and explain ideas- which is the cornerstone of a good developer.

**The Questions**

The first example focuses on application design and analysis.  Here in New York, we have MTA vending machines.  These are straightforward machines that let you buy either a prepaid or an all-you-can ride pass with credit or cash.  It&#8217;s a simple system, everyone in New York has probably used it, and is perfect system to talk system design.  A regular ATM machine or other common system also works well- you want the developer already have an understanding of the system so they can focus on application design.

_Design the system architecture for an MTA vending machine.  How would you break up the system into various components? How would the system work? What problems can occur? How can you deal with them?_

I get to see how well a candidate can come up with all the different things you&#8217;d need to know about a vending machine, and how thorough they are in their application design.  If they say &#8220;I&#8217;ll create a user input class and a payment processor&#8221; they&#8217;re not thinking through the issues. If they ramble on for ten minutes, they&#8217;re not organized with their thoughts or approach.  I want to know how well they understand tiers and class design- roles and responsibilities and program organization.  Don&#8217;t be afraid to ask what classes they would create to implement this system.

This question should focus on a system the user is already comfortable with.  Don&#8217;t have them gather requirements- it throws another dimension which can be difficult to maneuver, and takes away from the core point of system design.  It&#8217;s important to ease them into this interview approach, so they don&#8217;t get flustered or confused (however, it probably says a lot about them if they do).

Here&#8217;s a data modelling question inspired by Flickr:

_Design a data model for photo album software.  There should be photos, where each photo belongs to one, and only one album.  Albums belong to users.  Users can view other people&#8217;s photos if they are marked as public.  If a photo is private, no one can see the photo.  Each photo can be tagged using a simple &#8220;tagging&#8221; system.  Photos can have comments, which are linked to both the photo and the user who wrote the comment.  Finally, photos can belong to sets.  A set is a group of photos from various users.  A set is owned by a single user, but anyone can add a photo to a set._

This question works well on two fronts.  First, it obviously covers data modelling.  More importantly, it gauges the candidates ability to absorb information.  The question is explaining what we need from a business perspective- it doesn&#8217;t list any specific attributes.  The candidate should think and ask what we need to put in these tables.  The question also highlights entity relationships, to see how well the candidate can deal with joins and data organization.  Unlike the MTA question, I&#8217;m not approaching this as if they new what system I was talking about.  I could make this like the MTA question by saying, &#8220;Datamodel Flickr&#8221;.  I didn&#8217;t.  I&#8217;m looking how well they go from what the customer wants to an implementation- specifically, a data model.

Here&#8217;s a web service related question:

<p style="margin-left: 1.6pt;">
  <em>Let&#8217;s say I have two existing systems- an ordering system and a warehouse system.  The ordering system handles billing customers and fulfilling orders.  The warehouse system tracks inventory and sends shipments.  Currently, each night, orders are printed from the ordering system and faxed to the warehouse, where they are checked to see if they have any inventory.  If they are, they are fulfilled.  If not, they get put on hold and the warehouse calls the sales rep to have them update the ordering system to mark the order â€œon holdâ€ until the product comes in.  Your job is to make the ordering system and warehouse system talk to each other via web services.  What does the communication between the two system will look like?  What functions need to be exposed at which points? </em>
</p>

<p style="margin-left: 1.6pt;">
  There&#8217;s a part two:
</p>

<p style="margin-left: 1.6pt;">
  <em>A problem has occurred: Unfortunately, the connection between the two sites may be unreliable, but will never be down for more than a day.  Itâ€™s too cost prohibitive to upgrade the connection, and you&#8217;re asked if there&#8217;s anything you can do with the new service.  How would you design the system so a user can enter an order on a web page that will eventually get fulfilled by the warehouse?</em>
</p>

I like this because not only does it talk about vanilla web services, it also hints on extensibility.  Here, the candidate is asked to lay out a web service design and come up with some schemas.  If they can boil this down to a simple client/server service, great.  If they come up with a complex two way communication system bad.  But, after they laid out the initial design, they&#8217;re confronted with a major issue.  What do they need to do to deal with this problem given what the original solution?

The big kahuna question- everything rolled into one:

This one is mostly geared towards senior level programmers.  I was asked this question once in an interview and loved it- it was very draining, but in the end, it allowed me to show the company I was worth it.  It essentially wraps up all kinds of role playing questions into one big one.  Be warned- it also takes a while to do thoroughly.

Essentially, you, as the interviewer, sketch out a set of web pages on paper.  Do this before hand.  When I got asked the question it was an insurance site.  A CMS system, blog site, or project tracking system works well too.  For the insurance system, there were four pages: The account owner, with a bunch of the usual fields (name, address, city, SS#).  Then, there was a form for cars- an owner insured a car.  Make, model, year, etc.  Next, there was a form for insured drivers.  This could just be account owner, or members of their family.  Cars were linked to insured drivers, and offered a price for the insurance.  The owner could buy the insurance or not.  You need to have four or five hand drawn web pages, each with a form, all related.

So you walk the candidate through the hand drawn forms and what needs to happen, and you have them do numerous things.  They need a data model.  They need to design the application- data tier, business objects, web pages, whatever classes they need- see what they come up with.  They need to figure out how to get from one page to another and maintain state.  Sometimes I throw in a curve ball- in order to figure out the quote price, you need to call a long web service which could take ten to twenty seconds.  What&#8217;s the user experience like? How do you deal with this?  Finally, you tell them they&#8217;re running a dev team, and ask how they would break up the work to the different developers.  See how they approach this- it will give you a sense of well they know how to delegate and run a team.

**What You&#8217;re looking For
  
** 

Remember, with the phone interview done, you should have a solid understanding of the technical ability of the candidate.  But that&#8217;s not the most important quality of a candidate.  Google will tell me how to use ThreadPool.QueueUserWorkItem for multithreaded programming.  The real quality is knowing when I need multithreaded programming and knowing the difference between the ThreadPool API and asynchronous delegates. That&#8217;s what you&#8217;re looking for in these answers- the meat, not the special sauce.

You can also create your own questions depending on the work you do.  The point is to focus on a specific thing you&#8217;re looking for- data modelling,  SOA, application architecture, user interface design, business objects, or everything all in one- and try and stear the conversation in the general direction you want.  But let the candidate do most of the driving.

With each question, have a specific area of focus.  See what their toolbox is like if you want to know about implementation or programming.  If it&#8217;s about requirements, see how well the analyze the problem and cover the bases.  You know the problems you need to solve- so set up sandbox environments to see if they have the skills to solve your problems in various areas.  You decide how specific to get- and whether these are one, five or 20 minute questions.

If you&#8217;re not getting what they want, you may need to nudge them a little.  But be careful not to ask lead in questions.  If you set up a question like, &#8220;Okay, tell me how you would handle error in the MTA machine?&#8221; and they reply with, &#8220;Exceptions&#8221; they simply don&#8217;t get it.  If the struggle with the answer they probably don&#8217;t know what to do- you&#8217;ll easily spot this when you see it.

**What&#8217;s Next**

There&#8217;s a lot more to the in house interview- everything isn&#8217;t solved with open ended questions.  My next post will cover what else to ask- those personal, character building questions as well as how to structure the interview.
