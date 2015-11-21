---
title: Go Style Directory Layout for Scala with SBT
author: Michael
type: post
date: 2014-12-07
url: /2014/12/go-style-directory-layout-for-scala-with-sbt/
dsq_thread_id:
  - 3298324899
categories:
  - Programming
tags:
  - go
  - golang
  - sbt
  - tips
  - tricks
---
I&#8217;ve come to appreciate Go&#8217;s directory layout where _test_ and _build_ files are located side-by-side. This promotes a conscience testing priority. It also enables easy navigation to usage of a particular class/trait/object along with the implementation. After reading through the [getting-better-every-day sbt documentation][1] I noticed you can easily change default directories for sources, alleviating the folder craziness of default projects. Simply add a few lines to your build.sbt:

<pre class="syntax scala">//Why do I need a Scala folder? I don't!
//Set the folder for Scala sources to the root "src" folder
scalaSource in Compile := baseDirectory.value / "src"

//Do the same for the test configuration. 
scalaSource in Test := baseDirectory.value / "src"

//We'll suffix our test files with _test, so we can exclude
//then from the main build, and keep the HiddenFileFilter
excludeFilter in (Compile, unmanagedSources) := HiddenFileFilter || "*_test.scala"

//And we need to re-include them for Tests 
excludeFilter in (Test, unmanagedSources) := HiddenFileFilter
</pre>

Although breaking from the norm of java-build tools may cause confusion, if you like the way something works, go for it; don&#8217;t chain yourself to past practices. I never understood the class-to-file relationship of java sources, and I absolutely _hate_ navigating one-item folders. Thankfully Scala improved the situation, but the sbt maven-like defaults are still folder-heavy. IDEs make the situation easier, but I prefer simple text editors; and to paraphrase Dan North, &#8220;Your fancy IDE is a painkiller for your shitty language&#8221;.

 [1]: http://www.scala-sbt.org/0.13.2/docs/Howto/defaultpaths.html
