---
title: 'Book Review: Go Programming Blueprints, and the beauty of a language.'
author: Michael
type: post
date: 2015-03-20
url: /2015/03/book-review-go-programming-blueprints-and-the-beauty-of-a-language/
dsq_thread_id:
  - 3612792289
categories:
  - Programming
tags:
  - book review
  - go
  - language design
  - programming
  - scala
---
Just over two years ago my wife and I [traveled around Asia for several months)[thegreatbigadventure.tumblr.com]. I didn’t do any programming while I was gone [but I did a great deal of reading][1], gaining a new-found appreciation for programming and technology. I became deeply interested in Scala and Go for their respective approachs to statically typed languages. Scala for its functional programming aspects and Go for its refreshing and intentionally succinct approach to interfaces, types and its anti-inheritance. The criticism I most often here with Scala; that’s it too open, too free-for-fall in its paradigms is in stark contrast to the main criticisms I hear of Go: it’s too limiting, too constrained.

Since returning a majority of my time is focused on Scala, yet I still keep a hand in the Go cookie jar. Both languages are incredibly productive, and I appreciate FP the more I use it and understand it. Scala’s criticism is legitimate; it can be a chaotic language. However, my personal opinion is the language shouldn’t constrain you: it’s the discipline of the programmer to write code well, not the language. A bad programmer is going to destroy any language; a good programmer can make any code beautiful. More importantly, no language is magical. A language is a tool, and it’s up to the programmer to use it effectively.

Learning a language is more than just knowing how to write a class or function. Learning a language is about composing these together effectively and using the ecosystem around the language. Scala’s benefit is the ecosystem around the JVM; idiomatic Scala is contentious debate, as you have the functional programmers on one side and the more lenient anti-javaists on the other (Martin Odersky’s talk [Scala: The Simple Parts][2] is a great overview of where Scala shines). Go, on the other hand, is truly effective when you embrace its opinions and leverage its ecosystem: understanding imports and go get, writing small, independent modules, reusing these modules, embracing interfaces, and understanding the power of goroutines.

Last summer I had the great pleasure of being a technical reviewer for Mat Ryer’s [Go Programming Blueprints][3]. I’ve read a great deal of programming books in my career and appreciated Mat’s approach to showcasing the power and simplicity of Go. It’s not for beginners programmers, but if you have some experience, not even with Go, you can kick-start a working knowledge easily with Mat’s book. My favorite aspect is it explains how to write idiomatic Go to build applications. One example application composes discrete services and links them with bitly’s NSQ library, another uses a routing library on top of Go’s httpRequest handler. The book isn’t just isolated to web programs, there’s a section on writing CLI apps which link together with standard in and standard out. For those criticizing Go’s terseness Mat’s book exemplifies what you can do with those terse systems: write scalable, composable apps that are also maintainable and readable. The books shows why so many exciting new tools are written in Go: you can do a lot with little, and they compile to statically linked, minimal binaries.

As you develop your craft of writing code, you develop certain opinions on the way code should work. When your language is inline with your opinions, or you develop opinions based on the language, you are effectively using that language. If you are learning a new language, like Go, but still applying your existing opinions on how to develop applications (say, by wishing the language had Generics), you struggle. Worse, you are attempting to shape a new language to the one you know, effectively programming in the old language. You should embrace what the language offers, and honor its design decisions. Mat’s book shows how to apply Go’s design decisions effectively. The language itself will evolve and grow, but it will do it in a way that enhances and honors its design decisions. And if you still don’t like it, or Scala, well there’s always Rust.

 [1]: http://blog.michaelhamrah.com/2013/04/six-months-of-computer-science-without-computers/
 [2]: https://www.youtube.com/watch?v=ecekSCX3B4Q
 [3]: http://bit.ly/GoBb
