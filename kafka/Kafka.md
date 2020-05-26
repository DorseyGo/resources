# Kafka

## 1. Introduction

### 1.1 What is Kafka?

Kafka is a <font color="#aa0"><u>distributed streaming platform</u></font>,

- <font color="#aa0">publish and subscribe</font> to streams of records, similar to message queue or enterprise messaging system
- store streams of records in a <font color="#aa0">fault-tolerant durable</font> way
- <font color="#aa0">process</font> streams of records as they occurs

### 1.2 Why Kafka?

Kafka is used for two broad classes of applications,

- building <font color="#aa0">real-time streaming pipelines</font> that reliably get data between systems or applications
- building <font color="#aa0">real-time streaming</font> applications that <font color="#aa0">transforms or react</font> to the streams of data

### 1.3 Basics

#### 1.3.1 Topics & Logs

A topic is a category or feed name to which records are published. Topics in Kafka are always <u>multi-subscriber</u>.

For each topic, the Kafka cluster maintains a partitioned log.

Each partition is <font color="#aa0">ordered</font>, <font color="#aa0">immutable</font> sequence of records that is continually appended to - a structured commit log. The records in the partitions are each assigned a sequential id number called <font color="#aa0"><b><i>offset</i></b></font> that <font color="#aa0">uniquely</font> identifies each record within partition.

Kafka cluster durably persists all publised records - whether or not they have been consumed - using a configurable retention period.

The partitions in the log serves serval purposes,

- allow to <font color="#aa0">scale</font> beyond a size that will be fit on a single server
- acts as the unit of <font color="#aa0">parallelism</font>

#### 1.3.2 Distribution

The partitions of log are distributed over Kafka cluster.

Each partition is <font color="#aa0">replicated</font> across a configurable number of servers for fault-tolerance.

Each partition has one server acts as <font color="#aa0">"leader"</font> and zero or more servers which acts as <font color="#aa0">"followers"</font>,

- <u>Leader</u> - handles all read and write requests for the partition
- <u>Followers</u> - passively replicate the leader. one of followers will automatically become the new leader if leader detected to be failed.

#### 1.3.3 Producers

Producers <font color="#aa0">publish</font> data to the topics of their choice.

Producer is responsible for choosing which record to assign to which partition within the topic. Two fashions,

- round-robin
- according to some semantic partition function (e.g. based on some key in record)

#### 1.3.4 Consumers

Consumers labels themselves with a <i>consumer group</i> name, and each record publised to a topic is delivered to one consumer instance within each subscribing consumer group.

The way consumption is dividing up the partitions in the log over the consumer instances so that each instance is the <font color="#aa0">exclusive</font> consumer of a "fair share" of partitions at any point in time.

If new instances join the group they will take over some partitions from other members of the group; if an instance dies, its partitions will be distributed to the remaining instances.

Kafka only provides total order over records <font color="#aa0">within</font> a partition, <b><i>NOT</i></b> between different partitions in a topic.

#### 1.3.5 Multi-tenancy

Multi-tenancy is enabled by configuring <u>which topics can produce or consume data</u>. There is also operations support for quotas.

#### 1.3.6 Guarantees

Guarantees that Kafka gives,

- Messages sent by a producer to a particular topic partition will be appended in the order they sent
- A consumer instance sees records in the order they are stored in a log
- For a topic with replication factor N, we will tolerate up to N-1 server failures without losing any records committed in a log

## 2. 