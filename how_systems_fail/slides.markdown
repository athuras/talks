# How Systems Fail

[@athuras](https://www.twitter.com/athuras)


![This Is Fine](resources/this_is_fine.jpg)



## Overview

1. Timeseries Data
2. The Lambda Architecture
3. How to Fail Repeatedly
4. How to Fail Continuously
5. A Message of Hope



# Timeseries Data

![Generic Timeseries](resources/generic_timeseries.png)


![All The Things](resources/all_the_things.png)



## General Philosophy

1. Ingest over some period `T`
2. Summarize for `t >= T`
3. Serve
4. Repeat steps 1-3 until you run out of money



### In Practise

![DIAGRAM Offline Pipeline](resources/offline_pipeline_diagram.png)



### Problems

1. Data availability delays
2. Data processing delays
3. Data serving delays
4. Data Consistency and/or Corruption


### Standard Apology Line

> My deepest condolences CUSTOMER, we understand that you appreciate the ability to know how much money you're spending. We're currently investigating an issue with our reporting system.



## Solution: Be Faster


![Online Processing Meme](resources/fast_furious.jpg)



### General Process

1. Consume events in _realtime_
2. Summarize data in _realtime_
3. Serve in _realtime_
4. Repeat steps 1-3 until you run out of money


> "It is _faster_, thus it will _definitely_ work."
>  \- No one.


### In Practise

![DIAGRAM Online Pipeline](resources/online_pipeline_diagram.png)



### Constraints

1. Volatile Storage is fast, but expensive
2. Persistent Storage is Slow, or fast-but-read-only
3. Distributed Queues are Flaky
4. Streaming Technologies are immature, dangerous
5. "Performance Isolation"


![Performance Isolation in the Cloud](resources/performance_isolation.jpg)



## Solution: Do Both

![DIAGRAM Lambda](resources/lambda_architecture_diagram.png)


![Timeseries Both](resources/timeseries_overlap.png)
Without Merging


![Timeseries Reconciled](resources/timeseries_reconciled.png)
Reconciled



### Problems

1. Complexity
2. Dependency Hell
3. Duplicate work
4. Cost



# How to Fail Offline



## Lets Talk About MapReduce

Specifically, in pathological environments


This is a happy MapReduce job:

![DIAGRAM MR Good](resources/healthy_mr_job.png)


This is an unhappy MapReduce job:

![DIAGRAM MR Bad](resources/unhealthy_mr_job.png)



## Invariants

1. All mappers must start before reducers can start
2. The last mapper must end before reducers can end.


## Scenario

1. High Cluster Utilization
2. Huge number of huge mappers
3. Huge number of huger reducers


## Flow
1. Mappers start
2. Reducers start
3. Cluster manager marks trailing map tasks as failed
4. No mappers can start (cluster overrun)
5. Wait, forwever.


## "Solution"
Require that reducers can't start until shuffle is totally completed.
This makes things slow (wallclock time).



# How to Fail Online


## Lets talk about ~~Storm~~ Heron

Specifically, in pathological environments


This is your Logical Topology

![DIAGRAM Heron](resources/heron_topology.png)


This is your Physical Topology

![DIAGRAM Heron Phys](resources/heron_topology_physical.png)


This is your Physical Topology on a bad network

![DIAGRAM Dragons](resources/heron_dragons.svg)


## Invariants

1. System needs to agree on where each logical component resides
2. Components need to agree on the state of each intra-topology message


## Network Partitions


![DIAGRAM Network Partition](resources/network_partition.svg)
This is a classic Network Partition Diagram


![DIAGRAM Latency Distribution](resources/latency_distribution.png)
But messages are sneaky things


Most enterprise distributed systems assume network partitions are ~~rare~~ impossible.


Remember the invariants from MapReduce?

> The last mapper must end before reducers can end.


What happens to the 'pseudo mappers' when they can't deliver messages anymore? Or vanish? Or reappear?



## "Solutions"

1. Limit use of coordination features (aggregations, groups)
2. Minimise liklihood of Network Partition (centralise)



# A Message Of Hope

![Dealwithit Lambda](resources/dealwithit_lambda.png)



# Constraints

1. Volatile Storage is fast, but expensive
2. Persistent Storage is Slow, or fast-but-read-only
3. Distributed Queues are Flaky
4. Streaming Technologies are immature, dangerous
5. "Performance Isolation"


## Persistent Storage is Bad

BigTable, Cassandra beg to differ


## Distributed Queues are Bad

Kafka, PubSub, Kinesis. All solid contenders.


## Streaming Technologies are Bad

Kafka, Spark, Flink, MillWheel...


## Performance Isolation

Still a problem.

Either pay Google or Amazon to deal with this for you, or you're on your own.



## The Future Is Online

Any Questions?
