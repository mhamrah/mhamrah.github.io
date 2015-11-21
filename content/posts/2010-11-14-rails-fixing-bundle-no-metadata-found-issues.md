---
title: 'Rails: Fixing Bundle “No Metadata Found” issues'
author: Michael
type: post
date: 2010-11-14
url: /2010/11/rails-fixing-bundle-no-metadata-found-issues/
dsq_thread_id:
  - 527224519
categories:
  - Programming
---
In playing around with Rails this weekend, I ran into an annoying error when trying to set up some bundles- specifically with Webrat and Cucumber, which I found very odd:

<pre class="syntax bash">/Users/Michael/.rvm/rubies/ruby-1.9.2-p0/lib/ruby/1.9.1/rubygems/package/tar_input.rb:111:in
`initialize': No metadata found! (Gem::Package::FormatError)
</pre>

Removing the dependencies in my Gemfile fixed the issue, but obviously left me without Cucumber and Webrat gems.

Googling didn&#8217;t provide an immediate solution to my problem, which is why I&#8217;m writing this post. [An issue hidden in the Bundle Github tracker had a solution][1]: delete the cache directory in your Ruby&#8217;s gem directory. The problem isn&#8217;t necessarily specific to Webrat or Cucumber; the problem appears to be when the cache directory gets out of sync with the actual repository, and gems which should be installable cannot be found.

After deleting the cache, **bundle install** ran without error with my new Webrat and Cucumber gems.

 [1]: https://github.com/carlhuda/bundler/issuesearch?state=closed&q=metadata#issue/603
