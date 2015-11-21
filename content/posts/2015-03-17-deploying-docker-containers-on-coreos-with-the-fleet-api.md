---
title: Deploying Docker Containers on CoreOS with the Fleet API
author: Michael
type: post
date: 2015-03-17
url: /2015/03/deploying-docker-containers-on-coreos-with-the-fleet-api/
dsq_thread_id:
  - 3604097018
categories:
  - Programming
tags:
  - coreos
  - docker
  - fleet
---
I&#8217;ve been spending a lot more time with CoreOS in search of a docker-filled utopian PaaS dreams. I haven&#8217;t found quite what I&#8217;m looking for, but with some bash scripting, a little http, good tools and a lotta love I&#8217;m coming close. There are no shortage of solutions to this problem, and honestly, nobody&#8217;s really figured this out yet in an easy, fluid, turn-key type of way. You&#8217;ve probably read about CoreOS, Mesos, Marathon, Kubernetes&#8230; maybe even dug into Deis, Flynn, Shipyard. You&#8217;ve spun up a cluster, and are like&#8230; This is great, now what.

What I want is to go from an app on my laptop to running in a production environment with minimal fuss. I don&#8217;t want to re-invent the wheel; there are too many people solving this problem in a similar way. I like CoreOS because it provides a bare-bones docker runtime with a solid set of low-level tools. Plus, a lot of people I&#8217;m close with have been using it, so the cross-pollination of ideas helps overcome some hurdles.

One of these hurdles is how you launch containers on a cluster. I really like [Marathon&#8217;s][1] http api for Mesos, but I also like the simplicity of CoreOS as a platform. CoreOS&#8217;s distributed init system is [Fleet][2], which leverages systemd for running a process on a CoreOS node (it doesn&#8217;t have to be a container). It has some nice features, but having to constantly write similar systemd files and run fleetctl to manage containers is somewhat annoying.

Turns out, [Fleet has an http API][3]. It&#8217;s not quite as nice as Marathon&#8217;s; you can&#8217;t easily scale to N number of instances, but it does come close. There are a few examples of using the API to launch containers, but I wanted a more end-to-end solution that eliminated boilerplate.

## Activate the Fleet API

The Fleet API isn&#8217;t enabled out-of-the-box. That makes sense as the API is currently unsecured, so you shouldn&#8217;t enable it unless you have the proper VPC set up. [CoreOS has good documentation on getting the API running][4]. For a quick start you can drop the following yaml snippet into your cloudconfig&#8217;s units section:

<pre class="syntax yaml">- name: fleet.socket
  drop-ins:
    - name: 30-ListenStream.conf
      content: |
        [Socket]
        ListenStream=8080
        Service=fleet.service
        [Install]
        WantedBy=sockets.target
</pre>

## Exploring the API

With the API enabled, it&#8217;s time to get to work. The [API has some simple documentation][3] but offers enough to get started. I personally like the minimal approach, although I wish it was more feature-rich (it is v1, and better than nothing).

You can do a lot with curl, bash and jq. First, let&#8217;s see what&#8217;s running. All these examples assume you have a FLEET_ENDPOINT environment variable set with the host and port:

On a side note, environment variables are key to reuse the same functionality across environments. In my opinion, they aren&#8217;t used nearly enough. Check out the [twelve-factor app&#8217;s config section][5] to understand the importance of environment variables.

<pre class="syntax bash">curl -s $FLEET_ENDPOINT/fleet/v1/units | jq '.units[] | { name: .name, currentState: .currentState}'
</pre>

Sure, you can get the same data by running `fleetctl list-units`, but the http command doesn&#8217;t involve ssh, which can be a plus if you have a protected network, are are running from an application or CI server.

## Creating Containers

Instead of crafting a fleet template and running `fleetctl start sometemplate` , we want to launch new units via http. This involves PUTting a resource to the /units/ endpoint under the name of your unit (it&#8217;s actually /fleet/v1/units, it took me forever to find the path prefix). The Fleet API will build a corresponding systemd unit from the json payload, and the content closely corresponds to what you can do with [a fleet unit file][6].

The schema takes in a `desiredState` and an array of `options` which specify the `section`, `name`, and `value` for each line. Most Fleet templates follow a similar pattern as exemplified with [the launching containers with Fleet guide][7]:

  1. Cleanup potentially running containers 
  2. Pull the container
  3. Run the container
  4. Define X-Fleet parameters, like conflicts.

Again we&#8217;ll use curl, but writing json on the command line is really annoying. So let&#8217;s create a `unit.json` for our payload defining the tasks for CoreOS&#8217;s apache container:

<pre class="syntax json">{
  "desiredState": "launched",
  "options": [
    {
      "section": "Service",
      "name": "ExecStartPre",
      "value": "-/usr/bin/docker kill %p-i%"
    },
    {
      "section": "Service",
      "name": "ExecStartPre",
      "value": "-/usr/bin/docker rm %p-%i"
    },
    {
      "section": "Service",
      "name": "ExecStartPre",
      "value": "/usr/bin/docker pull coreos/%p"
    },
    {
      "section": "Service",
      "name": "ExecStart",
      "value": "/usr/bin/docker run --rm --name %pi-%i -p 80 coreos/%p /usr/sbin/apache2ctl -D FOREGROUND"
    },
    {
      "section": "Service",
      "name": "ExecStop",
      "value": "/usr/bin/docker stop %p-%i"
    },
    {
      "section": "X-Fleet",
      "name": "Conflicts",
      "value": "%p@*.service"
    }
  ]
}
</pre>

There&#8217;s a couple of things of note in this snippet:

  * We&#8217;re adding a &#8220;-&#8221; in front of the docker kill and docker rm commands of the ExecStartPre tasks. This tells to Fleet to continue if there&#8217;s an error; these tasks are precautionary to remove an existing phantom container if it will conflict with the newly launched one.
  * We&#8217;re using Fleet&#8217;s systemd placeholders %p and %i to replace actual values in our template with values from the template name. This provides a level of agnosticism in our template; we can easily reuse this template to launch different containers by changing the name. Unfortunately this doesn&#8217;t quite work in our example because it&#8217;s apache specific, but if you were running a container with an entry point command specified, it would work fine. You&#8217;ll also want to manage containers under your own namespace, either in a private or public registry. 

We can launch this file with curl:

<pre class="syntax bash">curl -d @unit.json -w "%{http_code}" -H 'Content-Type: application/json' $FLEETCTL_ENDPOINT/fleet/v1/units/apache@1.service
</pre>

If all goes well you&#8217;ll get back a `201 Created` response. Try running the `list units` curl command to see your container task.

We can run `fleetctl cat apache@1` to view the generated systemd unit:

<pre class="syntax bash">[Service]
ExecStartPre=-/usr/bin/docker kill %p-%I
ExecStartPre=-/usr/bin/docker rm %p-%i
ExecStartPre=/usr/bin/docker pull coreos/%p
ExecStart=/usr/bin/docker run --rm --name %pi-%i -p 80 coreos/%p /usr/sbin/apache2ctl -D FOREGROUND
ExecStop=/usr/bin/docker stop %p-%i

[X-Fleet]
Conflicts=%p@*.service
</pre>

Want to launch a second task? Just post again, but change the instance number from 1 to 2:

<pre class="syntax bash">curl -d @unit.json -w "%{http_code}" -H 'Content-Type: application/json' $FLEETCTL_ENDPOINT/fleet/v1/units/apache@2.service
</pre>

When you&#8217;re done with your container, you can simple issue a delete command to tear it down:

<pre class="syntax bash">curl -X DELETE -w "%{http_code}" $FLEET_ENDPOINT/fleet/v1/units/apache@1.service
</pre>

## Deploying New Versions

Launching individual containers is great, but for continuous delivery, you need deploy new versions with no downtime. The example above used systemd&#8217;s placeholders for providing the name of the container, but left the apache commands in place. Let&#8217;s use another CoreOS example container from the [zero downtime frontend deploys][8] blog post. This `coreos/example` container uses an entrypoint and tagged docker versions to go from a v1 to a v2 version of the app. Instead of creating multiple, similar, fleet unit files like that blog post, can we make an agnostic http call that works across versions? Yes we can.

Let&#8217;s conceptually figure out how this would work. We don&#8217;t want to change the json payload across versions, so the body must be static. We could use some form of templating or find-and-replace, but let&#8217;s try and avoid that complexity for now. Can we make due with the options provided us? We know that the %p parameter lets us pass in the template name to our body. So if we can specify the name and version of the container we want to launch in the name of the unit file we PUT, we&#8217;re good to go.

So we want to:

<pre class="syntax bash">curl -d @unit.json -w "%{http_code"} -H 'Content-Type: application/json' $FLEETCTL_ENDPOINT/fleet/v1/units/example:1.0.0@1.service
</pre>

I tried this with the above snippet, but replaced the pull and run commands above with the following:

<pre class="syntax json">{
      "section": "Service",
      "name": "ExecStart",
      "value": "/usr/bin/docker run --rm --name %p-%i -p 80 coreos/%p"
    },
    {
      "section": "Service",
      "name": "ExecStop",
      "value": "/usr/bin/docker stop %p-%i"
    },
</pre>

Unfortunately, this didn&#8217;t work because the colon, :, in example:1.0.0 make the name invalid for a container. I could forego the name, but then I wouldn&#8217;t be able to easily stop, kill or rm the container. So we need to massage the %p parameter a little bit. Luckily, bash to the rescue.

Unfortunately, systemd is a little wonky when it comes to scripting in a unit file. It&#8217;s relatively hard to create and access environment variables, you need fully-qualified paths, and multiple lines for arbitrary scripts are discouraged. After googling how exactly to do bash scripting in a systemd file, or why an environment variable wasn&#8217;t being set, I began to understand the frustration in the community on popular distros switching to systemd. But we can still make do with what we have by launching a `/bin/bash` command instead of the vanilla `/usr/bin/docker`:

<pre class="syntax json">{
  "desiredState": "launched",
  "options": [
    {
      "section": "Service",
      "name": "ExecStartPre",
      "value": "-/bin/bash -c \"APP=`/bin/echo %p | sed 's/:/-/'`; /usr/bin/docker kill $APP-%i\""
    },
    {
      "section": "Service",
      "name": "ExecStartPre",
      "value": "-/bin/bash -c \"APP=`/bin/echo %p | sed 's/:/-/'`; /usr/bin/docker rm $APP-%i\""
    },
    {
      "section": "Service",
      "name": "ExecStartPre",
      "value": "/usr/bin/docker pull coreos/%p"
    },
    {
      "section": "Service",
      "name": "ExecStart",
      "value": "/bin/bash -c \"APP=`/bin/echo %p | sed 's/:/-/'`; /usr/bin/docker run --name $APP-%i -h $APP-%i -p 80 --rm coreos/%p\""
    },
    {
      "section": "Service",
      "name": "ExecStop",
      "value": "/bin/bash -c \"APP=`/bin/echo %p | sed 's/:/-/'`; /usr/bin/docker stop $APP-%i"
    },
    {
      "section": "X-Fleet",
      "name": "Conflicts",
      "value": "%p@*.service"
    }
  ]
}
</pre>

and we can submit with:

<pre class="syntax bash">curl -X PUT -d @unit.json -H 'Content-Type: application/json'  $FLEET_ENDPOINT/fleet/v1/units/example:1.0.0@1.service
</pre>

More importantly, we can easily launch multiple containers of version two simultaneously:

<pre class="syntax bash">curl -X PUT -d @unit.json -H 'Content-Type: application/json'  $FLEET_ENDPOINT/fleet/v1/units/example:2.0.0@1.service
curl -X PUT -d @unit.json -H 'Content-Type: application/json'  $FLEET_ENDPOINT/fleet/v1/units/example:2.0.0@2.service
</pre>

and then destroy version one:

<pre class="syntax bash">curl -X DELETE -w "%{http_code}" $FLEET_ENDPOINT/fleet/v1/units/example:1.0.0@1.service
</pre>

## More jq and bash fun

Let&#8217;s say you do start multiple containers, and you want to cycle them out and delete them. In our above example, we&#8217;ve started two containers. How will we easily go from v2 to v3, and remove the v3 nodes? The marathon API has a simple &#8220;scale&#8221; button which does just that. Can we do the same for CoreOS? Yes we can.

Conceptually, let&#8217;s think about what we want. We want to select all containers running a specific version, grab the full unit file name, and then curl a DELETE operation to that endpoint. We can use the Fleet API to get our information, jq to parse the response, and the bash pipe operator with xargs to call our curl command.

Stringing this together like so:

<pre class="syntax bash">curl -s $FLEET_ENDPOINT/fleet/v1/units | jq '.units[] | .name | select(startswith("example:1.0.0"))' | xargs -t -I{} curl -s -X DELETE $FLEET_ENDPOINT/fleet/v1/units/{}
</pre>

jq provides some very powerful json processing. We are pulling out the name field, and only selecting elements which start with our specific app and version, and then piping that to xargs. The -I{} flag for xargs is a substitution trick I learned. This allows you to do string placements rather than pass the field as an argument.

## Conclusion

I can pretty much guarantee no matter what you pick to run your Docker PaaS, it won&#8217;t do exactly what you want. I can also guarantee that there will be a lot to learn: new apis, new commands, new tools. It&#8217;s going to feel like pushing a round peg in a square hole. But that&#8217;s okay; part of the experience is formulating opinions on how you want things to work. It&#8217;s a blend of learning the patterns and practices of a tool versus configuring it to work the way you want. Always remember a few things:

  * Keep It Simple
  * Think about how it should work conceptually
  * You can do a lot with command line. 

With an API-enabled CoreOS cluster, you can easily plug deployment of containers to whatever build flow you use: your laptop, a github web hook, jenkins, or whatever flow you wish. Because all the above commands are bash, you can replace any part with a bash variable and execute appropriately. This makes parameterizing these commands into functions easy.

 [1]: https://github.com/mesosphere/marathon
 [2]: https://github.com/coreos/fleet
 [3]: https://github.com/coreos/fleet/blob/master/Documentation/api-v1.md
 [4]: https://coreos.com/docs/launching-containers/config/fleet-deployment-and-configuration/
 [5]: http://12factor.net/config
 [6]: https://coreos.com/docs/launching-containers/launching/fleet-unit-files/
 [7]: https://coreos.com/docs/launching-containers/launching/launching-containers-fleet/
 [8]: https://coreos.com/blog/zero-downtime-frontend-deploys-vulcand/
