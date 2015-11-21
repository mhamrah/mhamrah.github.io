---
title: Weird Issue with rspec and authlogic-openid
author: Michael
type: post
date: 2009-05-24
url: /2009/05/weird-issue-with-rspec-and-authlogic-openid/
syntaxhighlighter_encoded:
  - 1
dsq_thread_id:
  - 527224579
categories:
  - Programming
tags:
  - rails rspec ruby authlogic
---
I&#8217;ve been playing around rspec in my application which also uses [AuthLogic&#8217;s OpenID][1].  When doing

<pre class="brush: bash; title: ; notranslate" title="">rake spec </pre>

I get the following, very weird error:

<pre class="brush: bash; title: ; notranslate" title="">(in /Users/Michael/r/wc)
/Users/Michael/.gem/ruby/1.8/gems/activerecord-2.3.2/lib/active_record/base.rb:1964:in `method_missing': undefined method `config' for #&lt;Class:0x263efdc&gt; (NoMethodError)
from /Library/Ruby/Gems/1.8/gems/authlogic-oid-1.0.3/lib/authlogic_openid/acts_as_authentic.rb:35:in `optional_fields='
from /Users/Michael/r/wc/app/models/user.rb:3
from /Library/Ruby/Gems/1.8/gems/authlogic-2.0.13/lib/authlogic/acts_as_authentic/base.rb:37:in `acts_as_authentic'
from /Users/Michael/r/wc/app/models/user.rb:2
from /Library/Ruby/Site/1.8/rubygems/custom_require.rb:31:in `gem_original_require'
from /Library/Ruby/Site/1.8/rubygems/custom_require.rb:31:in `require'
</pre>

As seen above I&#8217;m using version 1.0.3 of the source code.  For the life of me, I couldn&#8217;t figure out what was wrong.  What was even stranger was the code was working when running the site.  It turns out, config got changed to rw_config, and the gem repository at rubyforge hasn&#8217;t been updated to the latest 1.0.4 [which includes this change][2]:

<pre>Change from using config to the new rw_config. Requires the latest version 
of authlogic. config was too general of a name and was causing conflicts 
for people in their own projects.

</pre>

So I changed the acts\_as\_authentic.rb file so config is named rw_config, and everything worked fine.

I&#8217;d love to find a good reference on how rspec loads gems, and why this isn&#8217;t an issue when running the site.  I&#8217;m also interested in knowning how to pull content as a gem from github source (the openid gem isn&#8217;t at gems.github.com).

 [1]: http://github.com/binarylogic/authlogic_openid
 [2]: http://github.com/binarylogic/authlogic_openid/commit/8a8c7729cf5590636892535167a86fffd40650b4
