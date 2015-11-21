---
title: 'Testing Akka’s FSM: Using setState for unit testing'
author: Michael
type: post
date: 2014-03-21
url: /2014/03/testing-akkas-fsm-using-setstate-for-unit-testing/
dsq_thread_id:
  - 2473593219
categories:
  - Programming
tags:
  - akka
  - fsm
  - scala
---
I wrote [about Akka&#8217;s Finite State Machine][1] as a way to model a process. One thing I didn&#8217;t discuss was testing an FSM. Akka has [great testing support][2] and FSM&#8217;s can easily be tested using the [`TestFSMRef`][3] class.

An FSM is defined by its states and the data stored between those states. For each state in the machine you can match on both an incoming message and current state data. Our previous example modeled a process to check data integrity across two systems. We&#8217;ll continue that example by adding tests to ensure the FSM is working correctly. [These should have been before we developed the FSM][4] but late tests are (arguably) better than no tests.

It&#8217;s important to test combinations of messages against various states and data. You don&#8217;t want to be in a position to run through a state machine to the state you want for every test. Luckily, there&#8217;s a handy `setState` method to explicitly set the current state and data of the FSM. This lets you &#8220;fast forward&#8221; the FSM to the exact state you want to test against.

Let&#8217;s say we want to test a `DataRetrieved` message in the `PendingComparison` state. We also want to test this message against various `Data` combinations. We can set this state explicitly:

<pre class="syntax scala">"The ComparisonEngine" - {
  "in the PendingComparison state" - {
    "when a DataRetrieved message arrives from the old system" - {
      "stays in PendingComparison with updated data when no other data is present" in {
        val fsm = TestFSMRef(new ComparisonEngine())
        
        //set our initial state with setState
        fsm.setState(PendingComparison, ComparisonStatus("someid", None, None))

        fsm ! DataRetrieved("oldSystem", "oldData")

        fsm.stateName should be (PendingComparison)
        fsm.stateData should be (ComparisonStatus("someid", Some("oldData"), None))
      }
    }
  }
}
</pre>

It may be tempting to send more messages to continue verifying the FSM is working correctly. This will yield a large, unwieldy and brittle test. It will make refactoring difficult and make it harder to understand what _should_ be happening.

Instead, to further test the FSM, be explicit about the current state and what should happen next. Add a test for it:

<pre class="syntax scala">"when a DataRetrieved message arrives from the old system" - {
  "moves to AllRetrieved when data from the new system is already present" in {
    val fsm = TestFSMRef(new ComparisonEngine())
        
    //set our initial state with setState
    fsm.setState(PendingComparison, ComparisonStatus("someid", None, Some("newData")))

    fsm ! DataRetrieved("oldSystem", "oldData")
    fsm.stateName should be (AllRetrieved)
    fsm.stateData should be (ComparisonStatus("someid", Some("oldData"), Some("newData")))
  }
}
</pre>

By mixing in ImplicitSender or using TestProbes we can also verify messages the FSM should be sending in response to incoming messages.

Testing is an essential part of developing applications. Unit tests should be explicit and granular. For higher level orchestration integration tests, taking a black-box approach, provide ways to oversee entire processes. Don&#8217;t let your code become too unwieldy to manage: use the tools at your disposal and good coding practices to stay lean. Akka&#8217;s FSM provides ways of programming transitional behavior over time and Akka&#8217;s FSM Testkit support provides a way of ensuring that code works over time.

 [1]: http://blog.michaelhamrah.com/2014/01/programming-akkas-finite-state-machines-in-scala/
 [2]: http://doc.akka.io/docs/akka/snapshot/scala/testing.html
 [3]: http://doc.akka.io/api/akka/snapshot/index.html#akka.testkit.TestFSMRef
 [4]: http://blog.michaelhamrah.com/2013/02/embracing-test-driven-development-for-speed/
