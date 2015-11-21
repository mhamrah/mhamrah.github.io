---
title: Embracing Test Driven Development for Speed
author: Michael
type: post
date: 2013-02-04
url: /2013/02/embracing-test-driven-development-for-speed/
dsq_thread_id:
  - 1064020083
categories:
  - Programming
tags:
  - development
  - TDD
  - Testing
---
A few months ago I helped a developer looking to better embrace test driven development. The session was worthwhile and made me reflect on my journey with TDD.

Writing tests is one thing. Striving for full test coverage, writing tests first and leveraging integration and unit tests is another. Some people find writing tests cumbersome and slow. Others may ignore tests for difficult scenarios or code spikes. When first working with tests I felt the same way. Over time I worked through issues and my feeling towards TDD changed. The pain was gone and I worked more effectively.

TDD is about speed. Speed of development and speed of maintenance. Once you leverage TDD as a way to better produce code you&#8217;ve unlocked the promise of TDD: Code more, debug less.

<!--more-->

## Stay In Your Editor

How many times have you verified something works by firing up your browser in development? Too many times. You build, you wait for the app to start, you launch the browser, you click a link, you fill in forms, you hit submit. Maybe there&#8217;s a breakpoint you step through or some trace statements you output. How much time have you wasted going from coding to verifying your code works? Too much time.

Stay in your editor. It has everything you need to get stuff done. Avoid the context switch. Avoid repetitive typing. Have one window for your code and another for your tests. Even on small laptops you can split windows to have both open at once. Gary Bernhardt, in an excellent [Peepcode][1], shows how he runs specs from within vim. Ryan Bates, in his screencast [How I Test][2], only uses the browser for UI design. If you leave your editor you are wasting time and suffering a context switch.

Every language has some sort of continuous testing runtime. Detect a file change, run applicable tests. Take a look at [Guard][3]. Selenium and company are excellent browser testing tools. Jasmine works great for Javascript. Rspec and Capybara are a solid combination. Growl works well for notifications. By staying in your editor you are coding all that manual verification away. Once coded you can repeat indefinitely.

## Start with Tests

Test driven doesn&#8217;t mean test after. This may be the hardest rule for newcomers to follow. We&#8217;ve been so engrained to write code, to design classes, to focus on OOP. We know what we need to do. We just need to do it. Once code works we&#8217;ll then write tests to ensure it always works. I&#8217;ve done this bad practice myself.

When you test last you&#8217;re missing the _why_. _Customer gets welcome email after signing up_ means nothing without context. If you know _why_ this is needed you are in a better position to define your required tests and start shaping your code. The notification could be a simple acknowledgement or part of some intricate flow. If you know the _why_ you are not driving blind. The what you will build and the how you will build it will follow. If you code the other way around, testing later, you&#8217;re molding the problem to your solution. Define the problem first, then solve succinctly.

## Start with Failing Tests

One of my favorite newbie mistakes is when a developer writes some code, then writes a test, watches the test pass, then is surprised when the code fails in the browser. But the test passed!

Anyone can write a green test. It is the action of going from red to green which gives the test meaning. Something needs to work, it doesn&#8217;t. Red state. You change your code, you make it work. Green state. Without the red state first you have no idea how you got to a green state. Was it a bug in your test? Did you test the right thing? Did you forget to assert something? Who knows.

Combined with the _why_ going from red to green gives the code shape. You don&#8217;t need to over-think class design. The code you write has purpose: it implements a need to make something work that doesn&#8217;t. As your functionality becomes more complex, your code becomes more nimble. You deal with dependencies, spawning new tests and classes when cohesion breaks down. You stay focused on your goal: make something work. Combined with git commits you have a powerful history to branch and backtrack if necessary. As always, don&#8217;t be afraid to refactor.

## Testing First Safeguards Agile Development

Testing first also acts as a safeguard. Too often developers will pull work from a backlog prematurely. They&#8217;ll make assumptions, code to those assumptions, and have to make too many changes before release. If the first thing you do after pulling a story is ask yourself &#8220;how can I verify this works&#8221; you&#8217;re thinking in terms of your end-user. You&#8217;re writing acceptance tests. You understand what you need to deliver. BDD tools like [Cucumber][4] put this paradigm in the foreground. You can achieve the same effect with vanilla integration tests.

## Always Test Difficult Code

Most of the time not testing comes down to two reasons. The code is too hard to test or the code is not worth testing. There are other reasons, but they are all poor excuses. If you want to test code you can test code.

Code shouldn&#8217;t be too hard to test. Testing distributed, asynchronous systems is hard but still testable. When code is too hard to test you have the wrong abstraction. You&#8217;re API isn&#8217;t working. You aren&#8217;t adhering to SOLID principles. Your testing toolkit isn&#8217;t sufficient.

Static languages can rely on dependency injection to handle mocking, dynamic languages can intercept methods. Tools like [VCR][5] and Cassette can fake http requests for external dependencies. Databases can be tested in isolation or [faked][6]. Asynchronous code can be tricky to test but becomes easier when separating pre and post conditions (you can also block in unit tests to handle synchronization).

The code you don&#8217;t test, especially difficult code, will always bite you. Taking the time to figure out how to test will clean up the code and will give you incredible insight into how your underlying framework works.

## Always Test Your Code

I worked with a developer that didn&#8217;t write tests because the requirements, and thus code, were changing too much and dealing with the failing tests was tedious. It actually signified a red flag exposing larger issues in the organization but the point is a common one. Some developers don&#8217;t test because code may be thrown out or it&#8217;s just a spike and not worth testing.

If you&#8217;re not testing first because it&#8217;s a faster way to develop, realize that there is no such thing as throw away code (on the other hand, [all code is throw away code][7]). Mixing good, tested code with untested code creates technical debt. If you put a drop of sewer in a barrel of wine you will have a barrel of sewer. The code has no _why_. It may be just a spike but it could also turn out to be the next best thing. Then you&#8217;re left retrofitting unit tests, fitting a square peg in a round hole.

## Balancing Integration and Unit Tests

Once you start testing first a lot of pieces fall into place. The balance between integration and unit tests is an interesting topic when dealing with code coverage. There will be overlap in code coverage but not in terms of covered functionality.

Unit tests are the distinct pieces of your code. Integration tests are how those pieces fit together. You have a customer class and a customer page. The unit tests are the rules around the customer model or the distinct actions around the customer controller. The integration tests are how the end user interacts with those models top to bottom. [Pivotal Labs talks about changing state in cucumber steps][8] showing how integration tests monitor the flow of events in an application. Unit tests are for the discrete methods and properties which drive those individual events.

## Automate

Developing applications is much more than coding. Focusing on tools and techniques at your disposal will help you write code more effectively. Your IDE, command line skills, testing frameworks, libraries and development paradigms are as important as the code you right. They are your tools and become more powerful when used correctly.

 [1]: https://peepcode.com/products/play-by-play-bernhardt
 [2]: http://railscasts.com/episodes/275-how-i-test
 [3]: https://github.com/guard/guard
 [4]: http://cukes.info/
 [5]: https://www.relishapp.com/vcr/vcr
 [6]: https://github.com/nulldb/nulldb
 [7]: http://code.dblock.org/treat-every-line-of-code-as-if-its-going-to-be-thrown-away-one-day
 [8]: http://pivotallabs.com/cucumber-step-definitions-are-not-methods/
