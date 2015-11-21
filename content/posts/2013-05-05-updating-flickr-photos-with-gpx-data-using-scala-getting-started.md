---
title: 'Updating Flickr Photos with Gpx Data using Scala: Getting Started'
author: Michael
type: post
date: 2013-05-05
url: /2013/05/updating-flickr-photos-with-gpx-data-using-scala-getting-started/
dsq_thread_id:
  - 1264819408
categories:
  - Programming
tags:
  - flickr
  - scala
---
If you read this blog you know I&#8217;ve just returned from six months of travels around Asia, documented on our tumblr, [The Great Big Adventure][1] with photos on [Flickr][2]. Even though my camera doesn&#8217;t have a GPS, I realized toward the second half of the trip I could mark GPS waypoints and write a program to link that data later. I decided to write this little app in Scala, a language I&#8217;ve been learning since my return. The app is still a work in progress, but instead of one long post I&#8217;ll spread it out as I go along.

## The Workflow

When I took a photo I usually marked the location with a waypoint in my GPS. I accumulated a set of around 1000 of these points spread out over three gpx (xml) files. My plan is to:

  1. Read in the three gpx files and combine them into a distinct list.
  2. For each day I have at least one gpx point, get all of my flickr images for that data.
  3. For each image, find the waypoint timestamp with the least difference in time.
  4. Update that image with the waypoint data on Flickr.

## Getting Started

If you&#8217;re going to be doing anything with Scala, learning [sbt][3] is essential. Luckily, it&#8217;s pretty straightforward, but the documentation across the internet is somewhat inconsistent. As of this writing, [Twitter&#8217;s Scala School SBT Documentation][4], which I used as a reference to get started, incorrectly states that SBT creates a template for you. It no longer does, with the preferred approach to use [giter8][5], an excellent templating tool. I created [my own simplified version][6] which is based off of the excellently documented [template by Yuvi Masory][7]. Some of the versions in build.sbt are a outdated, but it&#8217;s worthwhile reading through the code to get a feel for the Scala and SBT ecosystem. The g8 project also contains a good working example of custom sbt commands (like g8-test). One gotcha with SBT: if you change your build.sbt file, you must call _reload_ in the sbt console. Otherwise, your new dependencies will not be picked up. For rubyists this is similar to running _bundle update_ after changing your gemfile.

## Testing

I&#8217;m a big fan of TDD, and strive for a test-first approach. It&#8217;s easy to get a feel for the small stuff in the scala repl, but orchestration is what programming is all about, and TDD allows you to design and throughly test functionality in a repeatable way. The two main libraries are [specs][8] (actually, it&#8217;s now [specs2][9]) and [ScalaTest][10]. I originally went with specs2. It was fine, but I wasn&#8217;t too impressed with the output and not thrilled with the matchers. I believe these are all customizable, but to get a better feel for the ecosystem I switched to ScalaTest. I like ScalaTest&#8217;s default output better and the flexible composition of testing styles (I&#8217;m using FreeSpec) and matchers (ShouldMatchers) provide a great platform for testing. Luckily, both specs2 and scalatest integrate with SBT which provides continuous testing and growl support, so you don&#8217;t need to fully commit to either one too early.

 [1]: http://thegreatbigadventure.tumblr.com
 [2]: http://flickr.com/hamrah
 [3]: http://scala-sbt.org
 [4]: http://twitter.github.io/scala_school/sbt.html
 [5]: https://github.com/n8han/giter8
 [6]: https://github.com/mhamrah/sbt.g8
 [7]: https://github.com/ymasory/sbt.g8
 [8]: https://code.google.com/p/specs/
 [9]: http://etorreborre.github.io/specs2/
 [10]: http://www.scalatest.org/
