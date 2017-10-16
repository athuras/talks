# How Systems Fail

[@athuras](https://www.twitter.com/athuras)


![This Is Fine](how_systems_fail/resources/this_is_fine.jpg)



## Overview

1. Timeseries Data
2. The Lambda Architecture
3. How to Fail Repeatedly
4. How to Fail Continuously
5. A Message of Hope



# Timeseries Data

![Generic Timeseries](how_systems_fail/resources/generic_timeseries.png)


![All The Things](how_systems_fail/resources/all_the_things.png)



## General Philosophy

1. Ingest over some period `T`
2. Summarize for `t >= T`
3. Serve
4. Repeat steps 1-3 until you run out of money



### In Practise

![DIAGRAM Offline Pipeline](how_systems_fail/resources/offline_pipeline_diagram.png)



### Problems

1. Data availability delays
2. Data processing delays
3. Data serving delays
4. Data Consistency and/or Corruption


### Standard Apology Line

> My deepest condolences CUSTOMER, we understand that you appreciate the ability to know how much money you're spending. We're currently investigating an issue with our reporting system.



## Solution: Be Faster


![Online Processing Meme](how_systems_fail/resources/fast_furious.jpg)



### General Process

1. Consume events in _realtime_
2. Summarize data in _realtime_
3. Serve in _realtime_
4. Repeat steps 1-3 until you run out of money


> "It is _faster_, thus it will _definitely_ work."
>  \- No one.


### In Practise

![DIAGRAM Online Pipeline](how_systems_fail/resources/online_pipeline_diagram.png)



### Problems

1. Volatile Storage is Expensive
2. Persistent Storage is Slow, or a write-only-store
3. Distributed Queues are Flaky
4. Online Topologies are very difficult to tune
5. "Performance Isolation"


![Performance Isolation in the Cloud](how_systems_fail/resources/performance_isolation.jpg)



## Solution: Do Both

![DIAGRAM Lambda](how_systems_fail/resources/lambda_architecture_diagram.png)


![Timeseries Both](how_systems_fail/resources/timeseries_overlap.png)
Overlay of the two systems


![Timeseries Reconciled](how_systems_fail/resources/timeseries_reconciled.png)
Reconciled



### Problems

1. Complexity
2. Dependency Hell
3. Duplicate work
4. Cost



# How to Fail at Batch Processing


## Lets Talk About MapReduce

Specifically, at its limit.


This is a healthy MapReduce phase:

![DIAGRAM MR Good](how_systems_fail/resources/good_mr.png)



This is an unhealthy MapReduce phase:

~[DIAGRAM MR Bad](how_systems_fail/resources/bad_mr.png)


## Invariants

1. All mappers must have started before reducers can start
2. The last mapper must end before reducers can end.

## Scenario

1. High Cluster Utilization
2. Huge number of huge mappers
3. Huge number of huger reducers


1. Mappers start
2. Reducers start
3. Cluster manager marks trailing map tasks as failed
4. No mappers can start (cluster overrun)
5. Wait, forwever.


# How to Fail at Online Processing

- Most enterprise distributed systems assume network partitions are rare.
- But what _is_ a network partition anyways?


Remember the invariants from MapReduce?

> The last mapper must end before reducers can end.

What happens to the 'pseudo mappers' when they can't delivery messages anymore?

