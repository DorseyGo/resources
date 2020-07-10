#### 4.5 The Consumer

The Kafka consume works by issuing <font color="#aa0">"fetch"</font> requests to the brokers leading the partitions it wants to consume.

The consumer specifies its offset in the log with each request and receives back a chunk of log beginning from that position. The consumer thus has significant control over the position and can <b><font color="#aa0">rewind</font></b> it to the re-consume data if need be.

##### Push VS. Pull

Pros and cons for <b>PUSH</b> and <b>PULL</b> approaches,

- the rate of data transferred
  - a push-based system has difficulty dealing with diverse consumers as the broker controls the <u><b>rate</b></u> at which data is transferred.
  - the goal is generally for consumer to be able to consume the <u>maximum possible rate</u>. A pull-based system has the nicer property that the consumer simply falls behind and catches up when it can.
- another advantage of a pull-based system is that it lends itself to <font color="#aa0">aggressive batching</font> of data sent to the consumer. A push-based system must choose to either send a request immediately or accumulate more data and then send it later without knowledge of whether the downstream consumer will be able immediately process it.

The deficiency of a native pull-based system is that if the broker has no data the consumer may end up polling in a tight loop, effectively busy-waiting for data to arrive. To avoid this we have parameters in our pull request that allow the consumer request to <font color="#aa0">block in a "long poll" waiting</font> until data arrives (and optionally waiting until a given number of bytes is available to ensure large transfer sizes).

##### Consumer Position

Keeping track of <i>what</i> has been consumed is, surprisingly, one of the key performance points of a messaging system.

In Kafka, topic is divided into a set of totally ordered <b><i>partitions</i></b>, <font color="#aa0">each of which</font> is consumed by exactly one consumer within each subscribing consumer group at any given time. This means that the position of a consumer in each partition is just a single integer, the offset of the next message to consume.

There is a side benefit of this decision. A consumer can deliberately <font color="#aa0"><i>rewind</i></font> back to an old offset and <font color="#aa0"><i>re-consume</i></font> data.