---
title: Getting Ruby 1.9, Readline, Rails, and Mysql all running on Snow Leopard
author: Michael
type: post
date: 2009-12-27
url: /2009/12/getting-ruby-1-9-readline-rails-and-mysql-all-running-on-snow-leopard/
dsq_thread_id:
  - 527224569
categories:
  - Programming
tags:
  - rails
  - ruby
  - snow leopard
---
In my never ending love/hate relationship with Ruby, Rails and my Mac I&#8217;ve finally gotten Ruby 1.9 up and running with Rails 2.3 and MySql 64 bit.  All on Snow Leopard.  There was an even a little detour with Readline.  If you&#8217;ve scoured other posts about Snow Leopard, Ruby, Rails and Mysql and ended up here I feel your pain.  I hope this helps you on your way. [Most of this info][1] is from other places which I&#8217;ve explained in a little (just a little) but more depth.

### Install XCode

You need XCode to do any of this, so install it.  If you&#8217;re upgrading to Snow Leopard, reinstall XCode so you get the correct c compiler.

### Your profile

Here&#8217;s the deal.  We&#8217;re going to install ruby 1.9 to your /usr/local directory.  It will dump stuff in /usr/local/bin and other stuff in /usr/local/lib.  Why here?  That&#8217;s where it goes.  The default install of ruby on Snow Leopard 1.8, lives in /System/Library/Frameworks/Ruby.framework/Versions/Current.  Current is really an alias (actually, symlink) to the 1.8 directory at the same level.  For some it may seem like a good idea to install ruby 1.9 here.  It&#8217;s not.  Just put in /usr/local like everyone else.

Because ruby 1.9 will live in our /usr/local you have to help out your terminal a little.  You have to tell it where to look for the bin of ruby 1.9.  So when you run &#8220;ruby&#8221; from terminal you get the 1.9 version in /usr/local, not the 1.8 version in the System Library.  That&#8217;s why you have to add a path to /usr/local in your profile.  Do this from the terminal:

<pre class="brush: bash; title: ; notranslate" title="">mate ~/.bash_profile
</pre>

This says, &#8220;open or create the file .bash\_profile, in the home directory (~/), using textmate&#8221;.  You can use any other editor if you know how- but if you did you probably wouldn&#8217;t need to read this.  So just buy- and use- textmate.  It&#8217;s a nice app.  Now, .bash\_profile is a file used by bash, aka the terminal app, for settings.  Some places you&#8217;ll see &#8220;mate .profile&#8221; instead.  [This will work too][2]&#8211; but if you have a .profile and .bash\_profile you may run in to problems.  Just have one, preferably .bash\_profile, and write this in it:

<pre class="brush: bash; title: ; notranslate" title="">export PATH=/usr/local/bin:/usr/local/sbin:/usr/local/mysql/bin:$PATH
</pre>

This appends our /usr/local/bin and sbin to the current list of directories to search for when trying to find out where all those little commands live which you type into the terminal.  Keen eyes may have noticed the mysql/bin thrown in there.  This is for later.  The :$PATH at the end is extremely important- it includes other paths which are included in other places. Once this is done then type &#8220;source .bash_profile&#8221; from terminal to load the changes.

### Try Installing Ruby

One way to install ruby is by using [MacPorts][3].  If you want to get a little more hands on, we&#8217;re going to pull the source down and build it ourselves.  MacPorts is probably the easiest option.  We&#8217;re not doing the easy option.

Note: Read all this first!  In the terminal, make sure you&#8217;re in your home directory by doing a simple &#8220;cd&#8221;.  Then, do this:

<pre class="brush: bash; title: ; notranslate" title="">mkdir src
cd src
</pre>

We&#8217;re creating a new directory called src for our source files, and moving into said directory.  Now we can get the code:

curl -O ftp://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.1-p376.tar.gz
  
tar xzvf ruby-1.9.1-p376.tar.gz
  
cd ruby-1.9.1-p376/

Curl is a nice little app to pull things from the internet.  The -O option simply names the local file the same as the remote file.  Ruby 1.9.1-p376 was the latest version as of this writing, but [In my never ending love/hate relationship with Ruby, Rails and my Mac I&#8217;ve finally gotten Ruby 1.9 up and running with Rails 2.3 and MySql 64 bit.  All on Snow Leopard.  There was an even a little detour with Readline.  If you&#8217;ve scoured other posts about Snow Leopard, Ruby, Rails and Mysql and ended up here I feel your pain.  I hope this helps you on your way. [Most of this info][1] is from other places which I&#8217;ve explained in a little (just a little) but more depth.

### Install XCode

You need XCode to do any of this, so install it.  If you&#8217;re upgrading to Snow Leopard, reinstall XCode so you get the correct c compiler.

### Your profile

Here&#8217;s the deal.  We&#8217;re going to install ruby 1.9 to your /usr/local directory.  It will dump stuff in /usr/local/bin and other stuff in /usr/local/lib.  Why here?  That&#8217;s where it goes.  The default install of ruby on Snow Leopard 1.8, lives in /System/Library/Frameworks/Ruby.framework/Versions/Current.  Current is really an alias (actually, symlink) to the 1.8 directory at the same level.  For some it may seem like a good idea to install ruby 1.9 here.  It&#8217;s not.  Just put in /usr/local like everyone else.

Because ruby 1.9 will live in our /usr/local you have to help out your terminal a little.  You have to tell it where to look for the bin of ruby 1.9.  So when you run &#8220;ruby&#8221; from terminal you get the 1.9 version in /usr/local, not the 1.8 version in the System Library.  That&#8217;s why you have to add a path to /usr/local in your profile.  Do this from the terminal:

<pre class="brush: bash; title: ; notranslate" title="">mate ~/.bash_profile
</pre>

This says, &#8220;open or create the file .bash\_profile, in the home directory (~/), using textmate&#8221;.  You can use any other editor if you know how- but if you did you probably wouldn&#8217;t need to read this.  So just buy- and use- textmate.  It&#8217;s a nice app.  Now, .bash\_profile is a file used by bash, aka the terminal app, for settings.  Some places you&#8217;ll see &#8220;mate .profile&#8221; instead.  [This will work too][2]&#8211; but if you have a .profile and .bash\_profile you may run in to problems.  Just have one, preferably .bash\_profile, and write this in it:

<pre class="brush: bash; title: ; notranslate" title="">export PATH=/usr/local/bin:/usr/local/sbin:/usr/local/mysql/bin:$PATH
</pre>

This appends our /usr/local/bin and sbin to the current list of directories to search for when trying to find out where all those little commands live which you type into the terminal.  Keen eyes may have noticed the mysql/bin thrown in there.  This is for later.  The :$PATH at the end is extremely important- it includes other paths which are included in other places. Once this is done then type &#8220;source .bash_profile&#8221; from terminal to load the changes.

### Try Installing Ruby

One way to install ruby is by using [MacPorts][3].  If you want to get a little more hands on, we&#8217;re going to pull the source down and build it ourselves.  MacPorts is probably the easiest option.  We&#8217;re not doing the easy option.

Note: Read all this first!  In the terminal, make sure you&#8217;re in your home directory by doing a simple &#8220;cd&#8221;.  Then, do this:

<pre class="brush: bash; title: ; notranslate" title="">mkdir src
cd src
</pre>

We&#8217;re creating a new directory called src for our source files, and moving into said directory.  Now we can get the code:

curl -O ftp://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.1-p376.tar.gz
  
tar xzvf ruby-1.9.1-p376.tar.gz
  
cd ruby-1.9.1-p376/

Curl is a nice little app to pull things from the internet.  The -O option simply names the local file the same as the remote file.  Ruby 1.9.1-p376 was the latest version as of this writing, but][4] for the latest release.  Tar xzvf simply unpacks the compressed download.

**Very helpful hint:** _If you ever are unsure about a command, simply type the command and &#8211;help.  As in, curl &#8211;help.  This is very helpful._

Next, we get to the good stuff.  First, run autoconf simply by typing &#8220;autoconf&#8221;:

<pre class="brush: bash; title: ; notranslate" title="">autoconf</pre>

[Autoconf][5] is a tool used for generating configuration scripts.  It&#8217;s important you run this.  Then, run the configuration script.  The thing which worked for me was:

<pre class="brush: bash; title: ; notranslate" title="">./configure --prefix=/usr/local/ --enable-shared --with-readline-dir=/usr/local
</pre>

This tells us to install ruby in the /usr/local directory, build a shared library for ruby, and use the readline installation found in /usr/local. **Very Important:** some users prefer adding a suffix to the ruby 1.9 install so it doesn&#8217;t interfere with the system install of ruby.  By adding the &#8211;program-suffix=19 option to configure you&#8217;ll append &#8220;19&#8221; to all commands, like &#8220;ruby19&#8243; and &#8220;gem19&#8243;.  This is a smart idea as it won&#8217;t interfere with the default ruby installation.  Using this technique there are ways to easily switch between ruby installations.  If you don&#8217;t care about 1.8, and just want the ease of typing &#8220;ruby&#8221; and getting the latest 1.9, omit the &#8211;program-suffix option.

If you run ./configure and get an error of:

_configure: WARNING: unrecognized options: &#8211;with-readline-dir_

You did not run autoconf.  Type it and run it, then run configure again.

Sometimes you&#8217;ll see the &#8211;enable-pthread option.  There seems to be some debate on whether this is a good idea.  I say omit it unless you know what you&#8217;re doing.  You can google for more info.  Feel free to explore and google other configure options- simply type &#8220;configure &#8211;help&#8221; to list them all.

Next we need to run:

<pre class="brush: bash; title: ; notranslate" title="">make</pre>

This builds all the source files needed for the install.  If you run this and get an error of:

readline.c: In function â€˜username\_completion\_proc_callâ€™:
  
readline.c:1159: error: â€˜username\_completion\_functionâ€™ undeclared (first use in this function)

You don&#8217;t have readline- or at least the proper version of readline- installed.  This is a problem.  Let&#8217;s get it.

### Readline

<pre class="brush: bash; title: ; notranslate" title="">cd ~/src
curl -O ftp://ftp.gnu.org/gnu/readline/readline-6.0.tar.gz
tar xzvf readline-6.0.tar.gz
cd readline-6.0
./configure --prefix=/usr/local
make
sudo make install
</pre>

You&#8217;ve now installed readline.  You may get a warning of _install: you may need to run ldconfig at_ at the end_._ Don&#8217;t worry about it.  At least I didn&#8217;t have to worry about it.

### Do everything again

By now, you know the drill.  Hop back to the ruby source code directory and try it again.  But if you ran make in the ruby install step and got errors, just run &#8220;make clean&#8221; to reset everything.  It&#8217;s a good idea.

<pre class="brush: bash; title: ; notranslate" title="">make clean
autoconf
./configure --prefix=/usr/local/ --with-readline-dir=/usr/local --enable-shared
make
sudo make install
</pre>

That should be it.  Hopefully you&#8217;re error free.  Typing:

<pre class="brush: bash; title: ; notranslate" title="">which ruby</pre>

should return:

_/usr/local/bin/ruby_

and typing &#8220;ruby -v&#8221; should return the version of ruby you&#8217;ve just downloaded.  Unless you used the &#8211;program-suffix option above, then it&#8217;s probably &#8220;ruby19 -v&#8221;

### **R****ails**

Now you can download and install rails.  Simply run:

<pre class="brush: bash; title: ; notranslate" title="">sudo gem update --system
sudo gem install rails
</pre>

### Mysql

Mysql is a piece of cake.  Simply grab the x86_64 [install package from the Mysql site.][6] Even though it&#8217;s for 10.5, it works fine on 10.6 (Snow Leopard).  Once this is done, you can build the mysql gem:

<pre class="brush: bash; title: ; notranslate" title="">sudo env ARCHFLAGS="-arch x86_64" gem install mysql -- --with-mysql-config=/usr/local/mysql/bin/mysql_config
</pre>

### Fin

Now, try to get everything running.  Go back to your home directory, create a rails app, and see if it works:

<pre class="brush: bash; title: ; notranslate" title="">cd ~/
rails playground --databse=mysql
cd playground
rake db:create
script/server
</pre>

Go to http://localhost:3000, check your environment settings, and you should see:

<table style="height: 164px;" width="256">
  <tr>
    <td>
      Ruby version
    </td>
    
    <td>
      1.9.1 (i386-darwin10.2.0)
    </td>
  </tr>
  
  <tr>
    <td>
      RubyGems version
    </td>
    
    <td>
      1.3.5
    </td>
  </tr>
  
  <tr>
    <td>
      Rack version
    </td>
    
    <td>
      1.0
    </td>
  </tr>
  
  <tr>
    <td>
      Rails version
    </td>
    
    <td>
      2.3.5
    </td>
  </tr>
  
  <tr>
    <td>
      Active Record version
    </td>
    
    <td>
      2.3.5
    </td>
  </tr>
  
  <tr>
    <td>
      Active Resource version
    </td>
    
    <td>
      2.3.5
    </td>
  </tr>
  
  <tr>
    <td>
      Action Mailer version
    </td>
    
    <td>
      2.3.5
    </td>
  </tr>
  
  <tr>
    <td>
      Active Support version
    </td>
    
    <td>
      2.3.5
    </td>
  </tr>
</table>

And voila, you&#8217;re done!

UPDATE:

If you use Textmate to develop with Rails, your Textmate Ruby path will point to the system&#8217;s 1.8 version, so you&#8217;ll get awakard issues of Gems not being available or other weird stuff when trying to run Ruby within Textmate (like when you want to RSpec tests).  This fix is simple: go to Textmate -> Preferences -> Advanced -> Shell Variables and add TM_RUBY with a value of /usr/local/bin/ruby and you&#8217;ll be good to go.

 [1]: http://wonko.com/post/how-to-compile-ruby-191
 [2]: http://hayne.net/MacDev/Notes/unixFAQ.html
 [3]: http://www.macports.org/
 [4]: http://www.ruby-lang.org/en/
 [5]: http://www.gnu.org/software/autoconf/
 [6]: http://dev.mysql.com/downloads/mysql/5.1.html#macosx-dmg
