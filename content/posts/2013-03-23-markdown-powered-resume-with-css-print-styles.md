---
title: Markdown Powered Resume with CSS Print Styles
author: Michael
type: post
date: 2013-03-23
url: /2013/03/markdown-powered-resume-with-css-print-styles/
dsq_thread_id:
  - 1159790958
categories:
  - Programming
tags:
  - markdown
  - resume
---
As much as I wish a LinkedIn profile could be a substitute for a resume, it&#8217;s not, and I needed an updated resume. My previous resume was done some time ago with InDesign when I was on a design-tools kick. It worked well, but InDesign isn&#8217;t the best choice for a straight forward approach to a resume and I was not interested in going back to word. So in honor of my friend Karthik&#8217;s [programming themed resume][1] I had an idea: program my resume. My requirements were simple:

  * Easy to edit: I should be able to update and output with minimal effort.
  * Easy to design: Something simple, but not boilerplate.
  * Export to Html and PDF: For easy distribution.

I&#8217;m a big fan of [Markdown][2] and happy to see the prevalence of Markdown across the web, however fragmented. I use Markdown to publish this blog and felt it would work well for writing a resume. The only problem is layout: you have minimal control over structural html elements which can make aspects of design difficult. For writing articles this isn&#8217;t a problem but when you need structural markup for CSS it can be limiting. Luckily I found [Maruku][3], a ruby-based markdown interpreter which supports [PHP Markdown Extra][4] and a [new meta-data syntax][5] for adding id, css, and div elements to a page. It does take away from Markdown&#8217;s simplicity but adds enough structure for design. Combined with CSS I had everything I needed to fulfill my requirements.

My [markdown resume][6] is on GitHub. I was surprised it rendered well with GitHub-Flavored Markdown despite the extraneous Maruku elements. I knew I was on the right track. Maruku lets you add your own stylesheets to the html output which I used for [posting online][7]. One simple command gets me from markdown to ready-to-publish html. Exactly what I wanted.

Markulu supports pdf output as well, but requires a heavy LaTex install which I wasn&#8217;t happy with. I also wasn&#8217;t impressed with the LaTex PDF output. Luckily there&#8217;s an easy alternative: printing to PDF. I used some [SASS media query overrides][8] on top of Html 5 Boilerplate&#8217;s default styles to control the print layout in the way I wanted. You can even specify page breaks and print margins via CSS. I favored Safari&#8217;s pdf output over Chrome&#8217;s for the sole reason Safari automatically embedded custom fonts in the final PDF.

At the end of the day I realized I probably didn&#8217;t need to add explicit divs to Markdown; I could have gotten the layout I wanted with just vanilla Markdown and CSS3 queries. I also could have a semantically better markup if I used HAML to add <section> tags instead of divs where appropriate, but HAML would have added a considerable amount of extraneous information to the markup. I&#8217;m also not sure editing the raw HAML text would have been as easy as Markdown.

At the end of the day, it&#8217;s all a tradeoff. GitHub flavored markdown, Markdown Here and other interpreters support fenced code blocks; I like the idea of adding fenced blocks to get <section> elements to get semantic correctness and layout elements in the html output. Unfortunately there&#8217;s no official Markdown spec and support is somewhat fragmented across various implementations, but [hopefully it will come together soon][9]. Until then, if you need it, you can always fork. Luckily I didn&#8217;t have to take it that far.

 [1]: http://kufli.blogspot.com/2013/02/evolution-of-my-resume-karthik.html
 [2]: http://daringfireball.net/projects/markdown/syntax
 [3]: https://github.com/bhollis/maruku
 [4]: http://michelf.ca/projects/php-markdown/extra/
 [5]: http://maruku.rubyforge.org/proposal.html
 [6]: https://github.com/mhamrah/mlh.com/blob/master/michael-hamrah-resume.md
 [7]: http://www.michaelhamrah.com/michael-hamrah-resume.html
 [8]: https://github.com/mhamrah/mlh.com/blob/master/scss/resume.scss
 [9]: http://www.codinghorror.com/blog/2012/10/the-future-of-markdown.html
