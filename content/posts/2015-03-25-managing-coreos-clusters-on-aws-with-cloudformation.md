---
title: Managing CoreOS Clusters on AWS with CloudFormation
author: Michael
type: post
date: 2015-03-25
url: /2015/03/managing-coreos-clusters-on-aws-with-cloudformation/
dsq_thread_id:
  - 3625180804
categories:
  - Programming
tags:
  - aws
  - cloudformation
  - coreos
  - deployment
  - devops
---
Personally, I find CloudFormation a somewhat annoying tool, yet I haven&#8217;t replaced it with anything else. Those json files can get so ugly and unwieldy. Alternatives exist; you can try an abstraction like [troposphere][1] or [jclouds][2], or ditch cfn completely with something like [terraform][3]. These are interesting tools but somehow I find myself sticking with the straight-up json approach, the aws cli, and some bash scripting: the pieces are already there, they just need to be strung together. In the end it&#8217;s not that bad, and there are some tools and techniques I&#8217;ve picked up which really help out. I recently applied these to managing CoreOS clusters with CFN, and wanted to share a simplified version of the approach.

[CoreOS provides a default CloudFormation template][4] which is a great start for cluster experimentation. But scaling out, where nodes are coming and going, can be disastrous for etcd&#8217;s quorum consensus if you&#8217;re not careful. You just don&#8217;t want to remove nodes from a formed etcd cluster. [CoreOS&#8217;s cluster documentation][5] has a section on production configuration: you want a core set of nodes for running central services, with various worker nodes for specific purposes. We can elaborate this with a short-list of requirements:

_**You want to tag sets of instances with specific roles so you can group dependencies and isolate apps when needed.**_ Although possible, it&#8217;s unrealistic to actually run any app on any node. More likely you want to group apps into front-facing and back-facing and treat those nodes differently. For instance, you could map the IP&#8217;s of front-facing nodes to a Route53 endpoint.

_**You want a cluster of heterogeneous instances for different workloads**_ Certain apps require certain characteristics. Even though you&#8217;re running everything in docker containers, you still want to have c4&#8217;s for compute-intensive loads, r3&#8217;s for memory-intensive loads, etc. Look at your applications and map them to a system topology. You can also scale these groups of instances differently, but you want to see your entire system as a whole: not as independent, discrete parts.

_**At some point, you&#8217;ll need to update the configuration of your instances. You want to do this surgically, without accidentally destroying your cluster**_. You may be one bad cfn update from relaunching an auto scaling group or misconfiguring an instance which causes a replacement. Just like normal instances you want to apply updates and reconfiguration of nodes in a sane, logical way. If you only had one cfn template for your entire cluster, it&#8217;s all or nothing. That&#8217;s not a choice we want to make.

CoreOS won&#8217;t let you forget about the underlying nodes; it just adds a little abstraction so you don&#8217;t need to deal with specific nodes as much.

I&#8217;m assuming you&#8217;re familiar with CloudFormation and the basics of a template. For our setup we&#8217;ll start with the [us-east-1 hvm CoreOS template][6] and modify it along the way. This template create a straight-up CoreOS cluster launched in an Auto Scaling Group, uses a LaunchConfig&#8217;s UserData to set some Cloud-Config settings. Like most templates you need a few parameters to launch. The non-default ones are your keypair and the etcd Discovery Url for forming the cluster. We are going to launch this stack with the CLI (who needs user interfaces?)

Let&#8217;s create a bash script, `coreos-cfn.sh`, to call our create stack (don&#8217;t forget to chmod +x). We need a DiscoveryUrl so we&#8217;ll get a new one in our script and pass it as a parameter to CFN.

<pre class="syntax bash">#!/bin/bash 

DISCOVERY_URL=`curl -s -w "\n" https://discovery.etcd.io/new`
#Check to make sure the above command worked, or exit
[[ $? -ne 0 ]] &#038;&#038; echo "Could not generate discovery url." &#038;&#038; exit 1

if [ -z "$COREOS_KEYPAIR" ]; then
  KEYPAIR=yourkey.pem
fi

# Create the CloudFormation stack
aws cloudformation create-stack \
    --stack-name coreos-test \
    --template-body file://coreos-stable-hvm.template \
    --capabilities CAPABILITY_IAM \
    --tags Key=Name,Value=CoreOS \
    --parameters \
        ParameterKey=DiscoveryURL,ParameterValue=${DISCOVERY_URL} \
        ParameterKey=KeyPair,ParameterValue=${KEYPAIR}
</pre>

The `-z $KEYPAIR` tests to see if there&#8217;s a keypair set as an environment variable; if not, it uses the specified one. If you run `coreos-cfn.sh` you should see the CLI spit out the ARN for the stack. Before we do that, let&#8217;s make two minor tweaks.

There are two key pieces of information we want to remember from this cluster: The DiscoveryUrl, so can access cluster state, and the AutoScalingGroup, so we can easily inspect instances in the future. Because the DiscoveryUrl is a parameter the aws cli will remember it for you. We need to add the auto scaling group as an output:

<pre class="syntax json">"Outputs": {
    "AutoScalingGroup" : {
      "Value": { "Ref": "CoreOSServerAutoScale" }
    }
  }
</pre>

After launching the cluster we can use the CLI and some jq to get back these parameters. It&#8217;s a simple built-in storage mechanism of AWS, and all you need is the original stack name:

<pre class="syntax bash"># Get back the DiscoveryURL: Describe the stack, select the parameter list
DISCOVERY_URL=`aws cloudformation describe-stacks --stack-name coreos-test | \
  jq -r '[.Stacks[].Parameters[]][] | select (.ParameterKey == "DiscoveryURL") | .ParameterValue'`

# Get back the auto-scaling-group-id
LEADER_ASG=`aws cloudformation describe-stacks --stack-name coreos-test | \
  jq -r '[.Stacks[].Outputs[]][] | select (.OutputKey == "AutoScalingGroup") | .OutputValue'`

echo "Discovery Url is $DISCOVERY_URL and Leader ASG is $LEADER_ASG"
</pre>

Why is this important? Because now we can either inspect the state of the cluster via the disovery url service, or query the ASG to inspect running nodes directly:

<pre class="syntax bash"># Query AWS for Leader Nodes
$aws ec2 describe-instances --filters Name=tag-value,Values=$LEADER_ASG | \
  jq '.Reservations[].Instances[].NetworkInterfaces[].PrivateIpAddress'

# Inspect the Discovery Url for nodes, trimming port. 
$ `curl -s $DISCOVERY_URL | jq '.node.nodes[].value[0:-5]'

# Taking the latter one step further, we can build an Etcd Peers string using Jq, xargs and tr
$ ETCD_PEERS=`curl -s $DISCOVERY_URL | jq '.node.nodes[].value[0:-5]' | xargs -I{}  echo "{}:4001" | tr "\\n" ","`
# Drop the last ,
$ ETCD_PEERS=${ETCD_PEERS%?}
</pre>

Armed with this information we are now able to spin up new CoreOS nodes and have it use our CoreOS leader cluster for management. The [CoreOS Cluster Architecture page][5] has the specific `cloud-config` settings which amount to:

  * Disable etcd, we don&#8217;t need it
  * Set etcd peer settings to a comma delimited list of nodes for Fleet, Locksmith
  * Set environment variables for fleet and etcd in start scripts

We&#8217;ll make the etcd peer list a parameter for our template. We can duplicate our leader template, replace the `UserData` portion of the `LaunchConfig` with the updated settings from the link above, and add `{ Ref: }` parameters where appropriate. Let&#8217;s also add a metadata parameter as well:

<pre class="syntax json">"Parameters": {
    "EtcdPeers" : {
      "Description" : "A comma delimited list of etcd endpoints to use for state management.",
      "Type" : "String"
    },
    "FleetMetadata" : {
      "Description" : "A comma delimited list of key=value attributes to apply for fleet",
      "Type" : "String"
    }
  }
</pre>

We can use the `Ref` functionality to pass these to our `UserData` script of the `LaunchConfig`:

<pre class="syntax json">//other config above
  "UserData" : { "Fn::Base64":
          { "Fn::Join": [ "", [
            "#cloud-config\n\n",
            "coreos:\n",
            "  fleet:\n",
            "    metadata: ", { "Ref": "FleetMetadata" }, "\n",
            "    etcd_servers: $", { "Ref": "EtcdPeers" }, "\n",
            "  locksmith:\n",
            "    endpoint: ", { "Ref": "EtcdPeers" }, "\n"
            ] ]
          }

// Other config below
</pre>

Finally we need a bash script which lets us inspect the existing stack information to pass as parameters to this new template. I also appreciate a CLI tool with a sane set of explicit flags. When I launch a secondary set of CoreOS nodes, I&#8217;d like something simple to set the name, type, metadata and where I want to join to:

<pre class="syntax bash">$ launch-worker-group.sh -n r3-workers -t r3.large -j coreos-test -m "instancetype=r3,role=worker"
</pre>

Bash has a flag-parsing abilities in its `getopts` function which we&#8217;ll simply use to set variables:

<pre class="syntax bash">#!/bin/bash

while getopts n:j:m:s: FLAG; do
  case $FLAG in
    n)  STACK_NAME=${OPTARG};;
    j)  JOIN=${OPTARG};;
    m)  METADATA=${OPTARG};;
    t)  INSTANCE_TYPE =${OPTARG};;
    [?])
      print >&#038;2 "Usage: $0 [ -n stack-name ] [ -j join to leader] [ -m fleet-metadata ] [ -t instance-type ]"
      exit 1;;
  esac
done

shift $((OPTIND-1))

# You can set defaults, too:
if [ -z $INSTANCE_TYPE ]; then 
  INSTANCE_TYPE ="m3.medium"
fi
</pre>

With this in place it&#8217;s just a matter of calling the AWS CLI with our new template and updated parameters. The only thing we&#8217;re doing differently than the original script is using CloudFormation&#8217;s json parameter functionality. This allows for more structured data in variables. Otherwise the comma-delimited list for etcd peers will throw off the CLI call.

<pre class="syntax bash">DISCOVERY_URL=`aws cloudformation describe-stacks --stack-name $JOIN | \
  jq -r '[.Stacks[].Parameters[]][] | select (.ParameterKey == "DiscoveryURL") | .ParameterValue'`
# Taking the latter one step further, we can build an Etcd 
# Peers string using jq, xargs and tr to flatten
ETCD_PEERS=`curl -s $DISCOVERY_URL | jq '.node.nodes[].value[0:-5]' | \
  xargs -I{}  echo "{}:4001" | tr "\\n" ","`

# Drop the last ,
ETCD_PEERS=${ETCD_PEERS%?}

 # Create the CloudFormation stack
 aws cloudformation create-stack \
    --stack-name STACK_NAME \
    --template-body file://coreos-worker-hvm.template \
    --capabilities CAPABILITY_IAM \
    --tags Key=Name,Value=CoreOS Key=Role,Value=Worker \
    --parameters "[
      { \"ParameterKey\":\"FleetMetadata\",\"ParameterValue\":\"${METADATA}\" },
      { \"ParameterKey\":\"InstanceType\",\"ParameterValue\":\"${INSTANCE_TYPE}\" },
      { \"ParameterKey\":\"EtcdPeers\",\"ParameterValue\":\"${ETCD_PEERS%?}\" },
      { \"ParameterKey\":\"KeyPair\",\"ParameterValue\":\"${KEYPAIR}\" }
    ]"
</pre>

And launch it! This will create a new stack for your worker nodes with whatever metadata you want, with whatever instance type you want.

There are a few ways to extend this. For one, we haven&#8217;t dealt with updating or destroying the stack. You can create separate shell scripts or combine them together with flags for determining which action to take. I prefer the latter as it keeps all related scripts in one file, but you can break out accordingly. You can use the AWS CLI and the Stack Name to query for private ip&#8217;s and update Route 53 accordingly, bypassing the need for an ELB.

You can do a lot with bash and other CLI tools like jq. You don&#8217;t need to scour GitHub for open source tools, or frameworks that have bells and whistles. The core components are there, you just need to glue them together. Yes, your scripts may get out of hand, but at that point it&#8217;s worth looking for alternatives because there&#8217;s probably a specific problem you need to solve. Remember, be opinionated and let those choices guide you. At some point in the future I may be raving about Terraform; friends say it&#8217;s a great tool, but it&#8217;s just not one that I need-or particularly want-to use now.

 [1]: https://github.com/cloudtools/troposphere
 [2]: https://jclouds.apache.org
 [3]: https://www.terraform.io
 [4]: https://coreos.com/docs/running-coreos/cloud-providers/ec2/
 [5]: https://coreos.com/docs/cluster-management/setup/cluster-architectures/
 [6]: https://s3.amazonaws.com/coreos.com/dist/aws/coreos-stable-hvm.template
