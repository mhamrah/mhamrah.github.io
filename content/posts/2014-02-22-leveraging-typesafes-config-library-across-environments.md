---
title: Using Typesafe’s Config for Scala (and Java) for Application Configuration
author: Michael
type: post
date: 2014-02-23
url: /2014/02/leveraging-typesafes-config-library-across-environments/
dsq_thread_id:
  - 2306006155
categories:
  - Programming
tags:
  - config
  - configuration
  - deployment
  - devops
  - runtimes
  - scala
---
I recently leveraged [Typesafe&#8217;s Config][1] library to refactor configuration settings for a project. I was very pleased with the API and functionality of the library.

The documentation is pretty solid so there&#8217;s no need to go over basics. One feature I like is the clear hierarchy when specifying configuration values. I find it helpful to put as much as possible in a reference.conf file in the /resources directory for an application or library. These can get overridden in a variety of ways, primarily by adding an application.conf file to the bundled output&#8217;s classpath. The [sbt native packager][2], helpful for deploying applications, makes it easy to attach a configuration file to an output. This is helpful if you have settings which you normally wouldn&#8217;t want to use during development, say using remote actors with akka. I find placing a reasonable set of defaults in a reference.conf file allows you to easily transport a configuration around while still overriding it as necessary. Otherwise you can get into copy and paste hell by duplicating configurations across multiple files for multiple environments.

## Alternative Overrides

There are two other interesting ways you can override configuration settings: using environment variables or java system properties. The environment variable approach comes in very handy when pushing to cloud environments where you don&#8217;t know what a configuration is beforehand. Using the ${?VALUE} pattern a property will only be set if a value exists. This allows you to provide an option for overriding a value without actually having to specify one.

Here&#8217;s an example in a conf file using substitution leveraging this technique:

    http {
     port = 8080
     port = ${?HTTP_PORT}
    }
    

We&#8217;re setting a default port of 8080. If the configuration can find a valid substitute it will replace the port value with the substitute; otherwise, it will keep it at 8080. The configuration library will look up its hierarchy for an HTTP\_PORT value, checking other configuration files, Java system properties, and finally environment variables. Environment variables aren&#8217;t perfect, but they&#8217;re easy to set and leveraged in a lot of places. If you leave out the ? and just have ${HTTP\_PORT} then the application will throw an exception if it can&#8217;t find a value. But by using the ? you can override as many times as you want. This can be helpful when running apps on Heroku where environment variables are set for third party services.

### Using Java System Properties

Java system properties provide another option for setting config values. The shell script created by sbt-native-packager supports java system properties, so you can also set the http port via the command line using the -D flag:

    bin/bash_script_from_native_packager -Dhttp.port=8081
    

This can be helpful if you want to run an akka based application with a different log level to see what&#8217;s going on in production:

    bin/some_akka_app_script -Dakka.loglevel=debug
    

Unfortunately sbt run doesn&#8217;t support java system properties so you can&#8217;t tweak settings with the command line when running sbt. The [sbt-revolver][3] plugin, which allows you to run your app in a forked JVM, does allow you to pass java arguments using the command line. Once you&#8217;re set up with this plugin you can change settings by adding your Java overrides after `---`:

    re-start --- -Dhttp.port=8081
    

### With c3p0

I was really excited to see that the [c3p0 connection pool library also supports Typesafe Config][4]. So you can avoid those annoying xml-based files and merge your c3p0 settings directly with your regular configuration files. I&#8217;ve migrated an application to a [docker][5] based development environment and used this c3p0 feature with [docker links][6] to set mysql settings:

    app {
     db {
      host = localhost
      host = ${?DB_PORT_3306_TCP_ADDR}
      port = "3306"
      port = ${?DB_PORT_3306_TCP_PORT}
     }
    }
    
    c3p0 {
     named-configs {
      myapp {
          jdbcUrl = "jdbc:mysql://"${app.db.host}":"${app.db.port}"/MyDatabase"
      }
     }
    }
    

When I link a mysql container to my app container with `--link mysql:db` Docker will inject the DB\_PORT\_3306\_TCP\_* environment variables which are pulled by the above settings.

### Accessing Values From Code

One other practice I like is having a single &#8220;Config&#8221; class for an application. It can be very tempting to load a configuration node from anywhere in your app but that can get messy fast. Instead, create a config class and access everything you need through that:

    object MyAppConfig {
      private val config =  ConfigFactory.load()
    
      private lazy val root = config.getConfig("my_app")
    
      object HttpConfig {
        private val httpConfig = config.getConfig("http")
    
        lazy val interface = httpConfig.getString("interface")
        lazy val port = httpConfig.getInt("port")
      }
    }
    

Type safety, Single Responsibility, and no strings all over the place.

### Conclusion

When dealing with configuration think about what environments you have and what the actual differences are between those environments. Usually this is a small set of differing values for only a few properties. Make it easy to change just those settings without changing&#8211;or duplicating&#8211;anything else. This could done via environment variables, command line flags, even loading configuration files from a url. Definitely avoid copying the same value across multiple configurations: just distill that value down to a lower setting in a hierarchy. By minimizing configuration files you&#8217;ll be making your life a lot easier.

If you&#8217;re developing an app for distribution, or writing a library, providing a well-documented configuration file ([spray&#8217;s spray-can reference.conf is an excellent example][7]) you can allow users to override defaults easily in a manner that is suitable for them and their runtimes.

 [1]: https://github.com/typesafehub/config
 [2]: https://github.com/sbt/sbt-native-packager
 [3]: https://github.com/spray/sbt-revolver
 [4]: http://www.mchange.com/projects/c3p0/#c3p0_conf
 [5]: docker.io
 [6]: http://docs.docker.io/en/latest/use/working_with_links_names/
 [7]: https://github.com/spray/spray/blob/master/spray-can/src/main/resources/reference.conf
