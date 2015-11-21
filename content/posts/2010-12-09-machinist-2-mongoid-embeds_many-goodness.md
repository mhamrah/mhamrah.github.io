---
title: Machinist 2 + Mongoid + Embeds_Many Goodness
author: Michael
type: post
date: 2010-12-10
url: /2010/12/machinist-2-mongoid-embeds_many-goodness/
dsq_thread_id:
  - 527410347
categories:
  - Programming
tags:
  - mongoid
  - rails
---
I had a heck of a time getting fixtures working with [Mongoid][1] when it came to a required embeds_many property.  No matter what I did, I kept getting an error: &#8220;_Access to the collection for_ XXX _is not allowed since it is an embedded document, please access a collection from the root document.&#8221;_

Then I stumbled upon [Machinist][2] v2 and the [machinist_mongo][3] gem which solved the problem.  And it had a nice API, to boot!

**Getting Started**

As of this post, you&#8217;ll need to pull the machinist_mongo gem directly from git and get the machinist2 branch.  That&#8217;s easy with Rails 3:

**gem &#8216;machinist\_mongo&#8217;, :git => &#8216;https://github.com/nmerouze/machinist\_mongo.git&#8217;, :require => &#8216;machinist/mongoid&#8217;, :branch => &#8216;machinist2&#8242;**

Next, run

<code class="syntax bash">bundle</code>

to update your gems.

I&#8217;m using RSpec, so I put my blueprints in the spec/support/ directory so they get automatically loaded. Let&#8217;s say I have a User class with a :username and an embeds_many :authentications property (as if you&#8217;re following the Railscasts episode on using Devise and Omniauth).  The blueprint will look like this:

<pre class="syntax ruby">Authentication.blueprint do
     uid { "user#{serial_number}" }
     provider { "machinist" }
end

User.blueprint do
     username { "user#{serial_number}" }
     authentications(1) { Authentication.make }
end
</pre>

My authentications blueprint sets a unique id using the #{serial_number} counter in machinist 2. Then I&#8217;m declaring an array of one item in my authentications array, and calling Authentication.make to load the Authentication blueprint. This essentially lazy loads the authentications property via the root document, which is exactly what Mongoid wants, as there&#8217;s no Authentications table or root-level document.

Now you can build away using Machinist 2&#8217;s User.make (for creating an object without saving it) or make! (which makes and saves the object).

 [1]: http://mongoid.org/
 [2]: https://github.com/notahat/machinist
 [3]: https://github.com/nmerouze/machinist_mongo
