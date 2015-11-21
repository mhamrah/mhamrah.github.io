---
title: Accelerate Team Development with your own SBT Plugin Defaults
author: Michael
type: post
date: 2014-10-13
url: /2014/10/accelerate-team-development-with-your-own-sbt-plugin-defaults/
dsq_thread_id:
  - 3112591399
categories:
  - Programming
tags:
  - sbt
  - scala
---
My team manages several Scala services built with SBT. The setup of these projects are very similar, from included plugins, dependencies, and build-and-deploy configurations. At first we simply copied and paste these settings across projects but as the number of services increased the hunt-and-change strategy became laborious. Time to optimize.

I heard of a few teams that created their own sbt plugins for default settings but couldn&#8217;t find information on how this looked. The recent change to [AutoPlugins][1] also didn&#8217;t help existing documentation. I found Will Sargent&#8217;s excellent post on [writing an sbt plugin][2] helpful but it wasn&#8217;t what I was looking for. I want a plugin which included other plugins and set defaults for those plugins. The goal is to &#8220;drop in&#8221; this plugin and automatically have a set of defaults: using [sbt-native-packager][3], a configured [sbt-release][4] and our nexus artifact server good-to-go.

## File Locations

As an sbt refresher anything in the `project/` folder relates to the build. If you want to develop your own plugin just for the current project you can simply add your .scala files to `project/`. If you want to develop your own plugin as a standalone project you put those files in the `src/` directory as usual. I mistakenly thought an sbt plugin project only required files in the `project/` folder. Silly me.

## SBT Builds

It&#8217;s important to note that the project folder&#8211;and the build itself&#8211;is separate from how your source code is built. SBT uses Scala 2.10, so anything in the `project/` folder will be built against 2.10 even if your project is set to 2.11. Thus when developing your plugin use Scala 2.10 to match sbt.

## Dependencies

Usually when you include a plugin you specify it in the `project/plugins.sbt`, right? But what if you&#8217;re developing a plugin that uses other plugins? Your code is in `src/` so it won&#8217;t pick up anything in `project/` as that only relates to your build. So you need to add whatever plugin you want as a `dependency` in your build so its available in within your project, just like any other dependency. But there&#8217;s a trick with sbt plugins. Originally I had the usual in `build.sbt`:

    libraryDependencies += "com.typesafe.sbt" % "sbt-native-packager" % "0.8.0-M2"
    

but kept getting unresolved dependency errors. This made no sense to me as the plugin is clearly available. It turns out if you want to include an sbt plugin as a project dependency you need to specify it in a special way, explicitly setting the sbt and scala version you want:

    libraryDependencies += sbtPluginExtra("com.typesafe.sbt" % "sbt-native-packager" % "0.8.0-M2", sbtV = "0.13", scalaV = "2.10")
    

With that, your dependency will resolve and you can use include anything under sbt-native-packager when developing your plugin.

## Specifying your Plugin Defaults

With your separate project and dependencies satisfied you can now create your plugin which uses other plugins and defaults settings specific to you. This part is easy and follows the usual documentation. Declare an object which extends AutoPlugin and override `projectSettings` or `buildSettings`. This class looks exactly like it would if you were setting things manually in your build.

For instance, here&#8217;s how we&#8217;d set the `java_server` archetype as the default in our plugin:

<pre class="code scala">package com.hamrah.plugindefaults

import sbt._
import Keys._
import com.typesafe.sbt.SbtNativePackager._

object PluginDefaults extends AutoPlugin {
 override lazy val projectSettings = packageArchetype.java_server
}
</pre>

You can concatenate any other settings you want to project settings, like scalaVersion, scalacOptions, etc.

## Using the Plugin

You can build and publish your plugin to a repo and include it like you would any other plugin. Or you can include it locally for testing by putting this in your sbt file:

    lazy val root = project.in( file(".") ).dependsOn( defaultPluginSettings )
    lazy val defaultPluginSettings = uri("file:///<full path to your plugin directory>")
    

Your default settings can be explicitly added to your project if not automatically imported with a simple:

    PluginDefaults.projectSettings
    //or
    settings = PluginDefaults.projectSettings // in a .scala file
    

## In Closing

As an FYI there could be better ways to do this. A lot of the above was trial and error, but works. If you have feedback or better suggestions please leave a comment!

 [1]: http://www.scala-sbt.org/0.13/docs/Plugins.html
 [2]: tersesystems.com/2014/06/24/writing-an-sbt-plugin
 [3]: https://github.com/sbt/sbt-native-packager
 [4]: https://github.com/sbt/sbt-release
