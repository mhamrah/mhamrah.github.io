---
title: First Class Function Example in Scala and Go
author: Michael
type: post
date: 2014-01-20
url: /2014/01/first-class-function-example-in-scala-and-go/
dsq_thread_id:
  - 2146881203
categories:
  - Programming
---
Go and Scala both make functions first-class citizens of their language. I recently had to recurse a directory tree in Go and came across the [Walk][1] function which exemplifies first-class functions. The Walk function talks a path to start a directory traversal and calls a function WalkFunc for everything it finds in the sub-tree:

<pre class="syntax go">func Walk(root string, walkFn WalkFunc) error </pre>

If you&#8217;re coming from the [Kingdom of Nouns][2] you may assume WalkFunc is a class or interface with a method for Walk to call. But that cruft is gone; WalkFunc is just a regular function with a defined signature given its own type, WalkFunc:

<pre class="syntax go">type WalkFunc func(path string, info os.FileInfo, err error) error
</pre>

Why is this interesting? I wasn&#8217;t surprised Go would have a built-in method for crawling a directory tree. It&#8217;s a pretty common scenario, and I&#8217;ve written similar code many times before. What&#8217;s uncommon about directory crawling is what you want to do with those files: open them up, move them around, inspect them. Separating the common from the uncommon is where first-class functions come into play. How much code have you had to write to just write the code you want?

Scala hides the OOP-ness of its underlying runtime by compile-time tricks, putting a first-class function like:

<pre class="syntax go">val walkFunc = (file: java.io.File) => { /* do something with the file */ }
</pre>

into a class of [Function1][3]. C# does something similar with its various [function classes][4] and delegate constructs. Go makes the interesting design decision of forcing function declarations outside of structs, putting an emphasis on stand-alone functions and struct extensibility. There are no classes in Go to encapsulate functions.

We can write a walk method for our walkFunc in Scala by creating a method which takes a function as a parameter (methods and functions have nuanced differences in Scala, but don&#8217;t worry about it):

<pre class="syntax scala">object FileUtil {
  def walk(file: File, depth: Int, walkFunc: (File, Int) => Unit): Unit = {
    walkFunc(file, depth)
    Option(file.listFiles).map(_.map(walk(_, depth + 1, walkFunc)))
  }
}
</pre>

In our Scala walk function we added a depth parameter which tracks how deep you are in the stack. We&#8217;re also wrapping the listFiles method in an Option to avoid a possible null pointer exception.

We can tweak our walkFunc and use our Scala walk function:

<pre class="syntax scala">import FileUtil._
val walkFunc = (path: File, depth: Int) => { println(s"$depth, ${path}") }
walk(new File("/path/to/dir"), 0, walkFunc)
</pre>

Because typing (File, Int) => Unit is somewhat obscure, type aliases come in handy. We can refactor this with a type alias:

<pre class="syntax scala">type WalkFunc = (File, Int) => Unit
</pre>

And update our walk method accordingly:

<pre class="syntax scala">def walk(file: File, depth: Int, walkFunc: WalkFunc): Unit = { ... }
</pre>

First class functions are powerful constructs making code flexible and succinct. If all you need is to call a function than pass that function as a parameter to your method. Just as classes have the [single responsibility principle][5] functions can have them too; avoid doing too much at once like file crawling and file processing. Instead pass a file processor call to your file crawling function.

 [1]: http://golang.org/pkg/path/filepath/#Walk
 [2]: http://steve-yegge.blogspot.com/2006/03/execution-in-kingdom-of-nouns.html
 [3]: http://www.scala-lang.org/api/current/index.html#scala.Function1
 [4]: http://msdn.microsoft.com/en-us/library/bb534960(v=vs.110).aspx
 [5]: http://en.wikipedia.org/wiki/Single_responsibility_principle
