---
title: Loving Vim
author: Michael
type: post
date: 2011-02-06
url: /2011/02/loving-vim/
dsq_thread_id:
  - 527224517
categories:
  - Programming
tags:
  - ide
  - vim
---
Vim has quickly become my go-to editor of choice for Windows, Mac and Linux. So far I&#8217;ve had about three months of serious Vim usage and I&#8217;m just starting to hit that vim-as-second-nature experience where the power really starts to shine. I&#8217;m shocked I&#8217;ve waited this long to put in the time to seriously learn it. Now that I&#8217;m past the beginner hump I wish I learned Vim long ago- when I tried Vim in the past, I just never got over that WTF-is-going-on-here frustration! Better late than never I suppose!

### Why I Like Vim &#8211; Mac

Coming from Visual Studio, I longed for a VS like experience for programming on my Mac, whether it&#8217;s html/css/js or for my recent focus on Ruby and Rails. I checked out both [Aptana][1] and [Eclipse][2] but quickly became frustrated- it was kind of like VS but not really and it was just too weird going back and forth. Plus, my biggest pet peave with development started to emerge: I wasn&#8217;t learning a language, I was learning a tool that abstracted the language away. There&#8217;s nothing that could be worse- once your tool hides the benefits of the underlying infrastructure, you&#8217;re missing the point, and you&#8217;ll usually be behind the curve because the language always moves faster than the support.

[TextMate][3] then became the go-to: it&#8217;s widely used, powerful, and there&#8217;s a lot of resources for learning. The simplified environment mixed with the command line really created a higher degree of fluidity, and I realized how nice it can be to develop outside an integrated environment. Textmate has its features- Command-T is slick, the project drawer is helpful, and the Rails support is great along with the other available bundles. But it lacked split windows which drove me crazy. There&#8217;s nothing more essential than split windows: I want to see my specs and code side-by-side. I want to see my html and js side-by-side. And you can&#8217;t do that with the Textmate. So I turned to [MacVim][4] and haven&#8217;t looked back.

### Why I Like Vim &#8211; Windows

Don&#8217;t get me wrong: I love me some Visual Studio. Visual Studio was my first IDE when programming professionally and my thought was &#8220;wow, I can really focus on building stuff rather than pulling my hair out with every build, every exception and every bug&#8221;. It was so much better than the emacs days of college. It&#8217;s my go-to for anything .NET, as it should be. But there are some text-editor needs that aren&#8217;t related to coding or .NET, and VS is too much of a beast to deal with for those things. First, for html/js/css editing that&#8217;s not part of a Visual Studio project, VS not great to work with. It&#8217;s annoying to be forced to create a Visual Studio project to house related content, especially when it&#8217;s already grouped together in the file system. Quickly checking out an html template or a js code samples becomes tedious when you just want to look around. The VS File Explorer is a step in the right direction, but it&#8217;s not there yet; I know there&#8217;s shell plugins for a &#8220;VS Project Here&#8221; shortcut but really? Is that necessary?

Then there&#8217;s the notepad issue. Notepad is barely an acceptable editor for checking out the occasional config file or random text file. Everyone knows how incredibly limited it can be, not to mention how much it sucks for large files. Pretty much everything about it sucks, actually, and everyone knows it. [Notepad++][5] is a nice alternative, but it&#8217;s no Vim. So after I got modestly comfortable with MacVim, I thought, why not do this on Windows too? So I started using [gVim][6] and haven&#8217;t looked back.

### Why I Like Vim &#8211; Linux

This blog runs WordPress on a Linux box hosted by Rackspace. Occasionally, I need to pop in via ssh, edit a config file, push some stuff, tweak some settings, etc. [Nano][7] was the lightweight go-to editor of choice, and works well. There are many differences between nano and Vim, the biggest being nano is a &#8220;modeless&#8221; editor while Vim&#8217;s power comes from the Normal, Insert, and Visual modes. Like always, stick with what works. But once you&#8217;re hooked on Vim, I&#8217;d be surprised how often you go back to Nano, especially when you get your configuration files synced up with source control, and Vim&#8217;s ubiquity on Linux.

### Why I Like Vim

There&#8217;s plenty of resources out there about Vim and what makes it great- a quick [google search][8] [will give you some great resources][9]. But I really appreciated Vim when I became comfortable with the following features:

#### Splits and Buffers

Splits are one of my favorite features- it allows you to view more than one file at once on the same screen. The power of splits allows you to have multiple vertically and horizontally split windows so you can see anything you want. There&#8217;s also tab support, but I find using splits and managing buffers a better way to cycle through files. Buffers allow you to keep files open and active but in the background, so you can quickly swap them into the current window in a variety of ways. It&#8217;s like having a stack of papers on a table while easily going to any page instantaneously. This is different than having files within a project, which would be like having those papers in a folder- buffers provide another level of abstraction allowing you to manipulate a set of content together. In summary, you have a set of papers in a folder (the current directory) and put them on the desk to work on (using buffers) and arrange them in front of you (using splits).

#### Modes

A major feature of Vim, which really sets it apart from other editors, is how it separates behavior into different modes- specifically, normal and insert modes. Insert mode is simple: it allows you to write text. But normal mode is all about navigation and manipulation: finding text, cutting lines, moving stuff around, substituting words, running commands, etc. It offers a whole new level of functionality including shell commands interaction for doing anything you would on the command line. With the plethora of plugins around you can pretty much do anything within Vim you could imagine- from simple editing, to testing, to source control management, to deployments. Modes break you free from a whole slew of Ctrl + whatever commands required in other editors, allowing for precision movement with a minimal set of keystrokes. The best analogy is you can &#8220;program your editing&#8221; in a way unmatched from any other editor.

#### Search and Replace (aka Substitute)

Vim&#8217;s Search and Substitute features make any old find and replace dialog box seem stupid. I&#8217;ve barely started unlocking the power of search and replace, but already, I wish every application behaved this way. In normal mode you can easily create everything from simple string searches to complex regex to find what your looking for in a couple of keystrokes. On top of that, it&#8217;s only a few more keystrokes to replace text. Because it&#8217;s all driven by key commands, you can easily change or alter what you&#8217;re doing without having to start an entire search and you never have that &#8220;context switch&#8221; of filling out a form in a dialog box. It&#8217;s all right in front of you. Highlighting allows you to see matches and you can lock in a search to easily jump between results. Viewing and editing configuration files is the real win for me over other editors: whatever the size, I can open a file and type a few keystrokes to go to exactly where I need to be, even if I&#8217;m not sure where to go. This is so much better than using notepad or even Visual Studio.

#### Source controlled configuration

[I put my vimfiles][10] on [github][11] so I can synchronize them across platforms. This offers an unparalleled level of uniformity across environments with minimal effort. A lot of people do this, and it&#8217;s helpful to see how others have configured their environment. You&#8217;ll pick up a lot of neat tidbits by reading people&#8217;s Vimfiles!

### Vim Across Platforms

Vim can run in either a gui window (like [MacVim][4] and [gVim][6]) or from a command line. These are two different executables with a slightly different feature set. Usually, you get a little more with a gui vim, especially around OS integration (like cutting and pasting text) while running shell commands are easier with command line vim. Gui Vim also offers better color support for syntax highlighting.

[MacVim][4] is the Vim app for the Mac. It has a really nice full screen mode and native Mac commands alongside the Vim ones. I love the [Peepopen][12] search plugin which is only available on the Mac (and can be used with Textmate). It&#8217;s a slick approach which is better than Textmate&#8217;s Command-T. I like running MacVim in full screen mode with the toolbar off to get the most screen real estate.

[gVim][6] is the Windows gui version of Vim, and I find it preferable over the command line Vim via cygwin. gVim has a shell extension which lets you open any file in gVim- set it as the default to avoid notepad. Note that Vim on Windows reads configuration files from the \_vimrc, \_gvimrc, and _vimfiles directory, which is different than the normal .vimrc and .vim location on other platforms. That hung me up when I was trying to sync configuration via git.

As for command line Vim, that&#8217;s probably what you&#8217;ll be using on Linux. In [Gary Bernhardt&#8217;s Play-by-Play Peepcode][13] video I learned a cool trick of running Vim via the command line, then using jobs to suspend (Ctrl-Z) to return to the command line, than going back to Vim via foreground (fg).

### Learning Vim

There are plenty of resources on the web for getting started with Vim. Steve Losh&#8217;s [Coming Home To Vim][14] is a great overview that points you to a lot of other helpful posts. I bought a subscription to [Peepcode][15] and picked up the [Smash Into Part II][16] episode. I didn&#8217;t watch Part I because I felt like it was too basic, but Part II has a lot of substantial content. Peepcode offers a high quality product so you probably can&#8217;t go wrong with getting both if you&#8217;d rather be safe than sorry. I also watched the [Gary Bernhardt Play by Play][13] which focuses a lot on using Vim with Rspec and Ruby, and use the [Peepopen][12] plugin for file search with Vim on my Mac.

Here are some tips to avoid the beginner frustration:

  * Take it slow. There&#8217;s a learning curve, but it&#8217;s worth it.
  * Don&#8217;t sweat plugins when you&#8217;re starting out. Yes, everyone says &#8220;use Pathogen, use Rails.vim, use xyz&#8221; and it&#8217;s absolutely correct. But it&#8217;s not essential when you&#8217;re starting out.
  * Take it one step at a time. Learn about Vim via tutorials and blogs so you know what&#8217;s there, but don&#8217;t try and do everything at once. Keep that knowledge in the back of your head, get comfortable with one thing, then move onto the next. You&#8217;ll just end up confusing yourself if you do everything at once.

Focusing on items in the following order will allow you to build on your knowledge:

  * editing text and searching, as that&#8217;s what you&#8217;ll be doing most
  * file management, like opening, saving, and navigating to files
  * other navigation, like jumping to lines or words and the power of hjkl (don&#8217;t use arrow keys!)
  * manipulation like replacing text, cutting and pasting
  * window and buffer management, including splits
  * start using and learning plugins to see where you can eliminate friction
  * start customizing your vimrc file to make the vim experience more comfortable now that you know the basics.

Good luck!

 [1]: http://www.aptana.com
 [2]: http://www.eclipse.org
 [3]: http://macromates.com
 [4]: http://code.google.com/p/macvim
 [5]: http://notepad-plus-plus.org/
 [6]: http://www.vim.org/download.php
 [7]: http://www.nano-editor.org/
 [8]: http://www.google.com/search?q=why+i+love+vim
 [9]: http://www.google.com/search?q=coming+home+to+vim
 [10]: https://github.com/mhamrah/vimfiles
 [11]: https://github.com/mhamrah/
 [12]: http://peepcode.com/products/peepopen
 [13]: http://peepcode.com/products/play-by-play-bernhardt
 [14]: http://stevelosh.com/blog/2010/09/coming-home-to-vim/
 [15]: http://www.peepcode.com
 [16]: http://peepcode.com/products/smash-into-vim-ii
