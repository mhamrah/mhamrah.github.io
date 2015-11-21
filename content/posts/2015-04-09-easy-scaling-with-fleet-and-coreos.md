---
title: Easy Scaling with Fleet and CoreOS
author: Michael
type: post
date: 2015-04-10
url: /2015/04/easy-scaling-with-fleet-and-coreos/
dsq_thread_id:
  - 3669461501
categories:
  - Programming
tags:
  - coreos
  - devops
  - fleet
  - scaling
---
One element of a successful production deployment is the ability to easily scale the number of instances your process is running. Many cloud providers, both on the PaaS and IaaS front, offer such functionality: AWS Auto Scaling Groups, Heroku&#8217;s process size, Marathon&#8217;s instance count. I was hoping for something similar in the CoreOS world. [Deis][1], the PaaS-on-CoreOS service, offers Heroku-like scaling, but I don&#8217;t want to commit to the Deis layer nor its build pack approach (for no other reason than personal preference). Fleet, CoreOS&#8217;s distributed systemd service, offers service templating, but you cannot say &#8220;now run three instances of service x&#8221;. Being programmers we can do whatever we want, and luckily, we&#8217;re only a little bash script away from replicating the &#8220;scale to x instances&#8221; functionality of popular providers.

[You&#8217;ll want to enable the Fleet HTTP Api][2] for this script to work. You can easily port this to the Fleet CLI, but I much prefer the http api because it doesn&#8217;t involve ssh, and provides more versatility into how and where you run the script.

Conceptually the flow is straightforward:

  * Given a process we want to set the number of running instances to some `desired_count`.
  * If `desired_count` is less than `current_count`, scale down.
  * If `desired_count` is more than `current_count`, scale up.
  * If they are the same, do nothing.

Fleet offers service templating so you can have a service unit named `my_awesome_app@.service` with specific copies named `my_awesome_app@1, my_awesome_app@2, my_awesome_app@N` representing specific running instances. Currently Fleet doesn&#8217;t offer a way to group these related services together but we can easily pattern match on the service name to control specific running instances. The steps are straightforward:

  * Query the Fleet API for all instances
  * Filter by all services matching the specified name
  * See how many instances we have running for the given service
  * Destroy or create instances using specific service names until we match the `desired_size`.

All of these steps are easily achievable with Fleet&#8217;s HTTP Api (or fleetctl) and a little bash. To give our script some context, let&#8217;s start with how we want to use the script. Ideally it will look like this:

<pre class="toolbar-overlay:false syntax bash">./scale-fleet my_awesome_app 5
</pre>

First, let&#8217;s set up our script `scale-fleet` and set the command line arguments:

<pre class="syntax bash">#!/bin/bash

FLEET_HOST=&lt;YOUR FLEET API HOST>

# You may want to consider cli flags 
SERVICE_NAME=$1
DESIRED_SIZE=$2
</pre>

Next we want to query the Fleet API and filter on all units with a prefix of `SERVICE_NAME` which have a process number. This will give us an array of units matching `my_awesome_app@1.service`, not the base template of `my_awesome_app@.service`. These are the units we will either add to or destroy as appropriate. The latest 1.5 version of jq supports regex expressions, but as of this writing 1.4 is the common release version, so we&#8217;ll parse the json response with jq, and then filter with grep. Finally some bash trickery will parse the result into an array we can loop through later.

<pre class="syntax bash"># Curls the API and filter on a specific pattern, storing results in an array
INSTANCES=($(curl -s $FLEET_HOST/fleet/v1/units | jq ".units[].name | select(startswith(\"$SERVICE@\"))" | grep '\w@\d\.service'))

# A bash trick to get size of array
CURRENT_SIZE=${#INSTANCES[@]}
echo "Current instance count for $SERVICE is: $CURRENT_SIZE"
</pre>

Next let&#8217;s scaffold the various scenarios for matching `CURRENT_SIZE` with `DESIRED_SIZE`, which boils down to some if statements.

<pre class="syntax bash">if [[ $DESIRED_SIZE = $CURRENT_SIZE ]]; then
  echo "doing nothing, current size is equal desired size"
elif [[ $DESIRED_SIZE &lt; $CURRENT_SIZE ]]; then
  echo "going to scale down instance $CURRENT_SIZE"
  # More stuff here
else 
  echo "going to scale up to $DESIRED_SIZE"
  # More stuff here
fi
</pre>

When the desired size equals the current size we don't need to do anything. Scaling down is easy, we simply loop, deleting the specific instance, until the desired and current states match. You can drop in the following snippet for scaling down:

<pre class="syntax bash">until [[ $DESIRED_SIZE = $CURRENT_SIZE ]]; do
    curl -X DELETE $FLEET_HOST/fleet/v1/units/${SERVICE}@${CURRENT_SIZE}.service

    let CURRENT_SIZE = CURRENT_SIZE-1
  done
  echo "new instance count is $CURRENT_SIZE"
</pre>

Scaling up is a bit trickier. Unfortunately you can't simply create a new unit from a template like you can with the fleetctl CLI. But you can do exactly what the fleetctl does: copy the body from the base template and create a new one with the specific full unit name. With the body we can loop, creating instances, until our current size matches the desired size. Let's walk it through step-by-step:

<pre class="syntax bash">echo "going to scale up to $desired_size"
 # Get payload by parsing the options field from the base template
 # And build our new payload for PUTing later
 payload=`curl -s $FLEET_HOST/fleet/v1/units/${SERVICE}@.service | jq '. | { "desiredState":"launched", "options": .options }'`

 #Loop, PUTing our new template with the appropriate name
 until [[ $DESIRED_SIZE = $CURRENT_SIZE ]]; do
   let current_size=current_size+1

   curl -X PUT -d "${payload}" -H 'Content-Type: application/json' $FLEET_HOST/fleet/v1/units/${SERVICE}@${CURRENT_SIZE}.service 
 done
 echo "new instance count is $CURRENT_SIZE"
</pre>

With our script in place we can scale away:

<pre class="syntax bash"># Scale up to 5 instances
$ ./scale-fleet my_awesome_app 5

# Scale down
$ ./scale-fleet my_awesome_app 3
</pre>

Because this all comes down to a simple bash script you can easily run it from a variety of places. It can be part of a parameterized Jambi job to scale manually with a UI, part of an [envconsul][3] setup with a key set in [Consul][4], or it can fit into a larger script that reads performance characteristics from some monitoring tool and reacts accordingly. You can also combine this with AWS Cloudformation or another cloud provider: if you're CPU's hit a certain threshold, you can scale the specific worker role running your instances, and have your `desired_size` be some factor of that number.

I've been on a bash kick lately. It's a versatile scripting language that easily portable. The syntax can be somewhat mystic, but as long as you have a shell, you have all you need to run your script.

The final, complete script is here:

 [1]: http://deis.io
 [2]: http://blog.michaelhamrah.com/2015/03/deploying-docker-containers-on-coreos-with-the-fleet-api/
 [3]: http://github.com/hashicorp/envconsul
 [4]: consul.io
