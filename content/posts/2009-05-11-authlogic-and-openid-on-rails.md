---
title: Authlogic and OpenID on Rails
author: Michael
type: post
date: 2009-05-11
url: /2009/05/authlogic-and-openid-on-rails/
syntaxhighlighter_encoded:
  - 1
dsq_thread_id:
  - 527224592
categories:
  - Programming
tags:
  - authlogic
  - openid
  - rails
---
So, I&#8217;m working on a Rails App and I want to use OpenID (and only OpenID) for authentication.  I was going to use Restful\_Authentication with the open\_id_authentication extension, but then I saw Ryan Bates&#8217; [Railscast][1] on [AuthLogic][2].  Authlogic has an [OpenID extention][3] which looked perfect for my needs, and Authlogic seemed like a great gem for authentication.  My goal was simple: I wanted to support, and only support, authentication via OpenID.  None of this username/password/salt stuff.

Now, I should warn you: I HAVE NO IDEA WHAT I AM DOING.  I&#8217;m just getting started with Rails so I hit a few bumps getting this working.  I learned a lot along the way, so here&#8217;s a quick rundown of my adventure with Authlogic and OpenID on Rails.

First, getting started with sample code is always a good idea.  Authlogic has an [example app][4] on github where you can check out how to use the gem.  After pulling down the code, I was surprised I couldn&#8217;t find anything relating to OpenID.  It turns out, the master repository doesn&#8217;t have the OpenID example.  But there&#8217;s a branch of the master example that does have the OpenID functionality.  Here&#8217;s how to get it:

First, clone the Authlogic example like so:

<pre class="brush: bash; title: ; notranslate" title="">git clone git://github.com/binarylogic/authlogic_example.git</pre>

Then, cd into the new Authlogic directory and get the OpenID fork like so:

<pre class="brush: bash; title: ; notranslate" title="">git checkout --track -b authlogic_with_openid origin/with-openid</pre>

Then, you can explore away.  My approach was simple: using the OpenID example as a guide, I&#8217;d start a new app and add OpenID support from Authlogic.

First, I need the required gems.  Authlogic and authlogic-oid are obvious, but I also need ruby-openid.  Authlogic-oid is built on the rails plugin open\_id\_authentication, which in turn is built on ruby-openid.

<pre class="brush: bash; title: ; notranslate" title="">sudo gem install ruby-openid
sudo gem install authlogic
sudo gem install authlogic-oid
</pre>

You can also set these in your environment.rb file, like so:

<pre class="brush: ruby; title: ; notranslate" title="">config.gem "authlogic"
config.gem "authlogic-oid", :lib =&gt; "authlogic_openid"
config.gem "ruby-openid", :lib =&gt; "openid"
</pre>

Note: I got thwarted when I tried running the app using the config.gem approach.  I kept getting an &#8220;Uninitialized constant Authlogic&#8221; in User_sessions.rb when I ran the app.  It sucked!  It took me a while to figure this out, but it was a horrible beginner mistake.  I was doing rake gems:install instead of sudo rake gems:install.  So when I ran my app, the proper gems weren&#8217;t in the right place.  Ouch!

Next, it&#8217;s time to install the open\_id\_authentication plugin which is required by Authlogic-oid.  It&#8217;s not available as a gem.

<pre class="brush: bash; title: ; notranslate" title="">script/plugin install git://github.com/rails/open_id_authentication.git</pre>

The next step is to create the necessary migration scripts for the open\_id\_authentication plugin.

<pre class="brush: bash; title: ; notranslate" title="">rake open_id_authentication:db:create</pre>

Another beginner mistake: I got a failure when I tried doing open\_id\_authentication:db:create before I installed the Authlogic gem.  Something about &#8220;acts\_as\_authenticated&#8221; wasn&#8217;t there.  So the order of installation is important!

Next, use the authlogic command to create the user_session model.  I&#8217;m prettying much following instructions from the Authlogic github pages at this point:

<pre class="brush: bash; title: ; notranslate" title="">script/generate session user_session
script/generate model user
</pre>

Another beginner mistake: if you&#8217;re making changes to your migration files without creating a new migration, make sure your schema is correct. I don&#8217;t know the best way to do this except by using rake db:drop, rake db:create, and rake db:migrate.  This was due to an error I was seeing in my view: &#8220;undefined method :openid\_identifier&#8221;.  I had a text\_field in my form for the openid field, and I had the openid_identifier field in my model.  The problem? It wasn&#8217;t in the database schema so Rails couldn&#8217;t do its thing to make it a property of the user model and render the textfield correctly.

From there, the views and controllers are pretty much the same as the example.  My user model is also pretty slim:

<pre class="brush: ruby; title: ; notranslate" title="">class CreateUsers &lt; ActiveRecord::Migration
def self.up
create_table :users do |t|
t.string :email, :null=&gt;false
t.string :persistence_token, :null=&gt; false
t.string :openid_identifier, :null=&gt; false
t.datetime :last_request_at
t.timestamps
end
add_index :users, :openid_identifier
add_index :users, :persistence_token
end
def self.down
drop_table :users
end
end
</pre>

Now, when I put everything together to run the app, I got some weird failures.  When trying to login using my OpenID, I got a nasty error:

<pre class="brush: bash; title: ; notranslate" title="">undefined local variable or method 'crypted_password_field'</pre>

Not having a crypted_pasword field made sense seeing how I wasn&#8217;t supporting login or password fields. But I wasn&#8217;t expecting _not_ having it to be an issue.  In fact, from the docs the only required field on the user model is :persistance_token. So what&#8217;s going on?

Well, it turns out the Authlogic OpenID extension is designed to work with a login/password by default.  I had to dig through the stack trace and source code, but was able to figure out what&#8217;s going on.  There&#8217;s a method called <span class="name">attributes_to_save which is responsible for persisting form fields across the OpenID process.  By default, </span><span class="name">attributes_to_save includes password related information.  It treats the crypted password and password salt fields a little differently, which causes a problem when you don&#8217;t have a :crypted_password attribute on your model.  The solution is simple: just override the method with one which doesn&#8217;t include the password fields.  The user model will look like this:</span>


<span class="name"> 

<pre class="brush: ruby; title: ; notranslate" title="">
class User &lt; ActiveRecord::Base
acts_as_authentic do |c|
def attributes_to_save # :doc:
attrs_to_save = attributes.clone.delete_if do |k, v|
[ :persistence_token, :perishable_token, :single_access_token, :login_count,
:failed_login_count, :last_request_at, :current_login_at, :last_login_at, :current_login_ip, :last_login_ip, :created_at,
:updated_at, :lock_version].include?(k.to_sym)
end
end
end
end
</pre>

<p>
  </span>
</p>

<p>
  The second mistake was because I got ahead of myself.  In the example app there&#8217;s code in the application.html.erb file rendering a login/register link or user info based on the current_user method in the application controller.  I was getting an error:
</p>

<pre class="brush: bash; title: ; notranslate" title="">unknown method 'logged_out?'</pre>

<p>
  <span class="n">which was occurring deep in the Authlogic codebase.  The problem was I didn&#8217;t go a good job copying everything I needed from the example files!  The authlogic example project used the</span>
</p>

<p>
  <span class="n">logout_on_timeout true</p> 
  
  <p>
    </span>
  </p>
  
  <p>
    <span class="n">filter on the UserSession method.  After digging through the documentation, this callback relies on the </span>
  </p>
  
  <p>
    <span class="n"> 
    
    <pre class="brush: ruby; title: ; notranslate" title="">
t.datetime :last_request_at
</pre>
    
    <p>
      </span>
    </p>
    
    <p>
      <span class="n">field on the user model, which I didn&#8217;t have at the time.  And not having this field was throwing everything off.  (The best part was the documentation clearly states you need that attribute on the model for the callback to work correctly.<br /> </span>
    </p>
    
    <p>
      <span class="n">Lesson learned: always know what you&#8217;re doing. (Also: don&#8217;t be afraid of source code).<br /> </span>
    </p>
    
    <p>
      The whole process has really made me appreciate Authlogic.  It&#8217;s very extensible and extremely easy to customize.  If you know what you&#8217;re doing it&#8217;s pretty slick- the best way to figure it out is by reading the documentation and playing around with some code.
    </p>
    
    <p>
      Good luck!
    </p>

 [1]: http://www.railscasts.com
 [2]: http://github.com/binarylogic/authlogic/tree/master
 [3]: http://github.com/binarylogic/authlogic_openid/tree/master
 [4]: http://github.com/binarylogic/authlogic_example/tree/master
