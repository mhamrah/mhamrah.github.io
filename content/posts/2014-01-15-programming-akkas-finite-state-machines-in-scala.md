---
title: Programming Akka’s Finite State Machines in Scala
author: Michael
type: post
date: 2014-01-16
url: /2014/01/programming-akkas-finite-state-machines-in-scala/
dsq_thread_id:
  - 2128112412
categories:
  - Programming
tags:
  - akka
  - design patterns
  - scala
---
Over the past few months my team has been building a new suite of services using Scala and Akka. An interesting aspect of Akka
  
we leverage is its [Finite State Machine][1] support. Finite State Machines
  
are a staple of computer programming although not often used in practice. A conceptual process can usually be represented with a finite state machine: there are a defined number of states with explicit transitions between states. If we have a vocabulary
   
around these states and transitions we can program the state machine.

A traditional implementation of an FSM is to check and maintain state explicitly via if/else conditions, use the state design pattern, or implement some other construct. Using Akka&#8217;s FSM support, which explicitly defines states and offers transition hooks, allows us to easily implement our conceptual model of a process. FSM is built on top of Akka&#8217;s actor model giving excellent concurrency controls so we can run many of these state machines simultaneously. You can implement your own FSM with Akka&#8217;s normal actor behavior with the _become_ method to change the partial function handling messages. However FSM offers some nice hooks plus data management in addition to just changing behavior.

As an example we will use Akka&#8217;s FSM support to check data in two systems. Our initial process is fairly simplistic but provides a good overview of leveraging Finite State Machines. Say we are rolling out a new system and we want to ensure data flows to both the old and new system. We need a process which waits a certain
  
amount of time for data to appear in both places. If data is found in both systems we will check the data for consistency,
  
if data is never found after a threshold we will alert data is out of sync.

Based on our description we have four states. We define our states using a sealed trait:

<pre class="syntax scala">sealed trait ComparisonStates
case object AwaitingComparison extends ComparisonStates
case object PendingComparison extends ComparisonStates
case object AllRetrieved extends ComparisonStates
case object DataUnavailable extends ComparisonStates
</pre>

Next we define the data we manage between state transitions. We need to manage an identifier with data from
  
the old and new system. Again we use a sealed trait:

<pre class="syntax scala">sealed trait Data
case object Uninitialized extends Data
case class ComparisonStatus(id: String, oldSystem: Option[SomeData] = None, newSystem: Option[SomeData] = None) extends Data
</pre>

A state machine is just a normal actor with the FSM trait mixed in. We declare our 

<pre class="brush: plain; title: ; notranslate" title="">ComparisonEngine</pre>

actor with FSM support,
  
specifying our applicable state and data types:

<pre class="syntax scala">class ComparisonEngine extends Actor with FSM[ComparisonStates, Data] {
}
</pre>

Instead of handling messages directly in a receive method FSM support creates an additional layer of messaging handling.
  
When using FSM you match on both message and current state. Our FSM only handles two messages: _Compare(id: Int)_ and
  
_DataRetrieved(system: String, someData: SomeData)_. You can construct your data types and messages any way
  
you please. I like to keep states abstract as we can generalize on message handling. This
  
prevents us from dealing with too many states and state transitions.

Let&#8217;s start implementing the body of our _ComparisonEngine_. We will start with our initial state:

<pre class="syntax scala">startWith(AwaitingComparison, Uninitialized)

when(AwaitingComparison) {
  case Event(Compare(id), Uninitialized) =>
    goto(PendingComparison) using ComparisonStatus(id)
}
</pre>

We simply declare our initial state is AwaitingComparison, and the only message we are willing to process is a Compare.
  
When we receive this message we go to a new state&#8211;PendingComparison&#8211;and set some data. Notice how we aren&#8217;t actually doing anything else?
  
A great aspect of FSM is the ability to listen on state transitions. This allows us to separate state transition logic from state transition
  
actions. When we transition from an initial state to a PendingComparison state we want to ask our two systems for data. We simply match
  
on state transitions and add our applicable logic:

<pre class="syntax scala">onTransition {
    case AwaitingComparison -> PendingComparison =>
      nextStateData match {
        case ComparisonStatus(id, old, new) => {
          oldSystemChecker ! VerifyData(id)
          newSystemChecker ! VerifyData(id)
        }
      }
    }
</pre>

_oldSystemChecker_ and _newSystemChecker_ are actors responsible for verifying data in their respective systems. These can be passed in to the FSM as constructor arguments, or you can have the FSM create the actors and supervise their lifecycle.
   
These two actors will send a DataRetrieved message back to our FSM when data is present. Because we are now in the _PendingComparison_
   
state we specify our new state transition actions against a set of possible scenarios:

<pre class="syntax scala">when(PendingComparison, stateTimeout = 15 minutes) {
  case Event(DataRetrieved("old", old), ComparisonStatus(id, _, None)) => {
    stay using ComparisonStatus(id, Some(old), None)
  }
  case Event(DataRetrieved("new", new), ComparisonStatus(id, None, _)) => {
    stay using ComparisonStatus(id, None, Some(new))
  }
  case Event(StateTimeout, c: ComparisonStatus) => {
    goto(IdUnavailable) using c
  }
  case Event(DataRetrieved(system, data), cs @ ComparisonStatus(_, _, _)) => {
    system match {
      case "old" => goto(AllRetrieved) using cs.copy(old = Some(data))
      case "new" => goto(AllRetrieved) using cs.copy(new = Some(data))
    }
  }
}
</pre>

Our snippet says we will wait 15 minutes for our systemChecker actors to return with data, otherwise, we&#8217;ll timeout and go to the unavailable state. Either the old
  
system or new system will return first, in which case, one set of data in our ComparisonStatus will be None. So we stay in the PendingComparison state until
   
the other system returns. If our previous pattern matches do not match, we know the current message we are processing is the final message. Notice how we don&#8217;t care how these actors are getting their data. That&#8217;s the responsibility of the child actors.
  
Once we have all our data,
   
so we go to the AllRetrieved state with the data from the final message.

There are a couple of ways we could have defined our states. We could have a state for the oldSystem returned or newSystem returned. I find it easier to
   
create a generic _PendingComparison_ state to keep our pattern matching for pending comparisons consolidated in a single partial function.

Our final states are pretty simple: we just stop our state machine!

<pre class="syntax scala">when(IdUnavailable) {
  case Event(_, _) => {
    stop
  }
}
when(AllRetrieved) {
  case Event(_, _) => {
    stop
  }
}
</pre>

Our last step is to add some more onTransition checks to handle our final states:

<pre class="syntax scala">case PendingComparison -> AllRetrieved =>
    nextStateData match {
      case ComparisonStatus(id, old, new) => {
        //Verification logic
     }
   }
 case _ -> IdUnavailable =>
   nextStateData match {
     case ComparisonStatus(id, old, new) => {
      //Handle timeout
      }
   }
</pre>

We don&#8217;t care how we got to the _AllRetrieved_ state; we just know we are there and we have the data we need. We can offload our verification logic
  
to another actor or inline it within our FSM as necessary.

Implementing processing workflows can be tricky involving a lot of boilerplate code. Conditions must be checked, timeouts handled, error handling implemented.
  
The Akka FSM approach provides a foundation for implementing workflow based processes on top of Akka&#8217;s great supervision support. We create a ComparisonEngine
  
for every piece of data we need to check. If an engine dies we can supervise and restart. My favorite feature is the separation of what causes a state transition
  
with what happens during a state transition. Combined with isolated behavior amongst actors this creates a cleaner, isolated and composable application to manage.

 [1]: http://doc.akka.io/docs/akka/2.2.3/scala/fsm.html
