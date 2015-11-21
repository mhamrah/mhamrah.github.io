---
title: Akka Clustering with SBT-Docker and SBT-Native-Packager
author: Michael
type: post
date: 2014-06-19
url: /2014/06/akka-clustering-with-sbt-docker-and-sbt-native-packager/
dsq_thread_id:
  - 2779000308
categories:
  - Programming
tags:
  - akka
  - docker
  - sbt
---
Since my last post on [akka clustering with docker containers][1] a new plugin, [SBT-Docker][2], has emerged which allows you to build docker containers directly from SBT. I&#8217;ve updated my [akka-docker-cluster-example][3] to leverage these two plugins for a smoother docker build experience.

## One Step Build

The approach is basically the same as the previous example: we use SBT Native Packager to gather up the appropriate dependencies, upload them to the docker container, and create the entrypoint. I decided to keep the start script approach to &#8220;prep&#8221; any environment variables required before launching. With SBT Docker linked to Native Packager all you need to do is fire

<pre class="brush: plain; title: ; notranslate" title="">docker
</pre>

from sbt and you have a docker container ready to launch or push.

## Understanding the Build

SBT Docker requires a dockerfile defined in your build. I want to pass in artifacts from native packager to docker. This allows native packager to focus on application needs while docker is focused on docker. Docker turns into just another type of package for your app.

We can pass in arguments by mapping the appropriate parameters to a function which returns the Dockerfile. In build.spt:

<pre class="syntax scala">// Define a dockerfile, using parameters from native-packager
dockerfile in docker &lt;&lt;= (name, stagingDirectory in Universal) map {
  case(appName, stageDir) =>
    val workingDir = s"/opt/${appName}"
    new Dockerfile {
      //use java8 base box
      from("relateiq/oracle-java8")
      maintainer("Michael Hamrah")
      //expose our akka port
      expose(1600)
      //upload native-packager staging directory files
      add(stageDir, workingDir)
      //make files executable
      run("chmod", "+x", s"/opt/${appName}/bin/${appName}")
      run("chmod", "+x", s"/opt/${appName}/bin/start")
      //set working directory
      workDir(workingDir)
      //entrypoint into our start script
      entryPointShell(s"bin/start", appName, "$@")
    }
}
</pre>

### Linking SBT Docker to SBT Native Packager

Because we&#8217;re relying on Native Packager to assemble our runtime dependencies we need to ensure the native packager files are &#8220;staged&#8221; before docker tries to upload them. Luckily it&#8217;s easy to create dependencies with SBT. We simply have docker depend on the native packager&#8217;s stage task:

<pre class="syntax scala">docker &lt;&lt;= docker.dependsOn(com.typesafe.sbt.packager.universal.Keys.stage.in(Compile))
</pre>

### Adding Additional Files

The last step is to add our start script to the native packager build. Native packager has a `mappings` key where we can add files to our package. I kept the start script in the docker folder and I want it in the bin directory within the docker container. Here's the mapping:

<pre class="syntax scala">mappings in Universal += baseDirectory.value / "docker" / "start" -> "bin/start"
</pre>

With this setting everything will be assembled as needed and we can package to any type we want. Setting up a cluster with docker is [the same as before][1]:

<pre class="syntax bash">docker run --name seed -i -t clustering
docker run --name c1 -link seed:seed -i -t clustering
</pre>

It's interesting to note SBT Native Packager also has docker support, but it's undocumented and doesn't allow granular control over the Dockerfile output. Until SBT Native Packager fully supports docker output the SBT Docker plugin is a nice tool to package your sbt-based apps.

 [1]: http://blog.michaelhamrah.com/2014/03/running-an-akka-cluster-with-docker-containers/
 [2]: https://github.com/marcuslonnberg/sbt-docker
 [3]: https://github.com/mhamrah/akka-docker-cluster-example
