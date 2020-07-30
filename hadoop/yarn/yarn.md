# 1. Architecture

The followings form the data-computation framework:

- <b><i>ResourceManager</i></b> - the ultimate authority that <font color="#aa0">arbitrates resource</font> among all the applications in the system
- <b><i>NodeManager</i></b> - the per-machine framework agent who is responsible for <font color="#aa0">containers, monitoring their resource usage (CPU, memory, disk, network) and reporting</font> the status to the ResourceManager/scheduler.

![architecture](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/yarn_architecture.gif)

The <font color="#aa0">ResourceManager</font> has two main components:

- <b><u>Scheduler</u></b> - responsible for <font color="#a00">allocating resources</font> to the various running applications subject to familiar constraints of <font color="#aa0">capacities, queues</font> etc. It performs its scheduling function based on the <font color="#aa0">resource requirements</font> of the application; it does so based on the abstract notion of a resource <i>Container</i> which incorporates elements such as <font color="#aa0">memory, CPU, disk, network</font> etc.
- <b><u>ApplicationsManager</u></b> - is responsible for <font color="#a00">accepting job-submissions, negotiating the first container for executing the application specific ApplicationMaster and provides the service for restarting the ApplicationMaster container on failure</font>. 

The per-machine <b><u>ApplicationMaster</u></b> has responsibility of <font color="#a00">negotiating appropriate resource containers from the Scheduler, tracking their status and monitoring for progress</font>.

# 2. Commands

## 2.1 User Commands

| CMD                      | Description                                          |
| ------------------------ | ---------------------------------------------------- |
| yarn app [options]       | prints application(s) report/kill application/manage |
| yarn container [options] | prints container reports                             |
| yarn logs [options]      | dump the container log                               |

# 3. Scheduler

## 3.1 CapacityScheduler

The <font color="#aa0">CapacityScheduler</font> is designed to run Hadoop applications as a <font color="#aa0">shared, multi-tenant cluster</font> in an operator-friendly manner while <font color="#aa0">maximizing the throughput</font> and <font color="#aa0">the utilization</font> of the cluster.

The <font color="#aa0">CapacityScheduler</font> provides a stringent set of limits to ensure that a single application or user or queue <font color="#aa0">cannot consume <b>disproportionate</b> amount of resources</font> in the cluster. Also, the <font color="#aa0">CapacityScheduler</font> provides <font color="#aa0">limits</font> on initialized and pending applications from a single user and queue to ensure <font color="#aa0">fairness</font> and <font color="#aa0">stability</font> of the cluster.

The primary abstraction provided by the <font color="#aa0">CapacityScheduler</font> is the concept of <font color="#aa0"><i>queues</i></font>.

### 3.1.1 Features

Following features:

- <b><u>Hierarchical Queues</u></b> - supported to ensure resources are <font color="#aa0">shared</font> among the sub-queues of an organization before other queues are allowed to use free resources, thereby providing more <font color="#aa0">control and predictability</font>.
- <b><u>Capacity Guarantees</u></b> - Queues are allocated a <font color="#a00">fraction</font> of the capacity of the grid in the sense that a certain capacity of resources will be at their disposal.
- <b><u>Security</u></b> - Each queue has strict <font color="#a00">ACLs</font> which controls which users can submit applications to individual queues.
- <b><u>Elasticity</u></b> - Fee resources can be allocated to any queue <font color="#a00">beyond</font> its capacity. 
- <b><u>Multi-tenancy</u></b> - comprehensive set of limits are provided to prevent a single application, user and queue from <font color="#a00">monopolizing</font> resources of the queue or the cluster as a whole to ensure that the cluster isn't overwhelmed.
- <b><u>Operability</u></b>
  - <b>Runtime Configuration</b> - the queue <font color="#aa0">definitions and properties</font> such as capacity, ACLs can be changed, at runtime, by administrators in a <font color="#aa0">secure manner</font> to minimize disruption to users. Administrators can <i>add additional queues</i> at runtime, but queues <b>CANNOT</b> be <i>deleted</i> at runtime unless the queue is <b>STOPPED</b> and has no pending/running apps.
  - <b>Drain Configuration</b> - administrators can <i>stop</i> queues at runtime to ensure that while existing applications run to completion, no new applications can be submitted.
- <b><u>Resource-based Scheduling</u></b> - support for <font color="#aa0">resource-intensive</font> applications, where-in a application can optionally specify <font color="#aa0">higher</font> resource-requirements than the default, thereby accommodating applications with differing resource requirements.
- <b><u>Queue Mapping interface based on Default or User defined placement Rules</u></b> - allows users to map a job to a specific queue based on some default placement rules.
- <b><u>Priority Scheduling</u></b> - allows applications to be <font color="#aa0">submitted and scheduled</font> with different priorities, only <b>FIFO</b> ordering policy supported currently.
- <b><u>Absolute Resource Configuration</u></b> - specify absolute resources to a queue instead of providing percentage based values.
- <b><u>Dynamic Auto-Creation and Management of Leaf Queues</u></b> - supports auto-creation of <font color="#aa0">Leaf queues</font> in conjunction with <font color="#aa0">queue-mapping</font> with currently supports <font color="#aa0">user-group</font> based queue mappings for application placement to a queue.

## 3.2 FairScheduler

# 4. Yarn Tuning

3 phases to Yarn tuning:

- <b><u>Cluster configuration</u></b>, where you configure your hosts.
- <b><u>Yarn configuration</u></b>, where you quantify memory and vcores.
- <b><u>MapReduce configuration</u></b>, where you allocate minimum and maximum resources for specific map and reduce tasks.

## 4.1 Cluster Configuration

In this phase, you define the worker host configuration and cluster size for your Yarn implementation.

Following steps consists of cluster configuration:

- <font color="#a00"><u>Worker host configuration</u></font> - define the configuration for a <font color="#aa0">single</font> worker host computer in your cluster.

- <font color="#a00"><u>Worker host planning</u></font> - <font color="#aa0">allocate resources</font> on each worker machine.

  | service                | CPU (cores) | Memory (MB) | Notes                                                        |
  | ---------------------- | ----------- | ----------- | ------------------------------------------------------------ |
  | Cloudera Manager Agent | 1           | 1024        | Allocate 1 GB for cloudera manager                           |
  | HDFS DataNode          | 1           | 1024        | allocation for the HDFS DataNode: default 1 vcore and 1 GB   |
  | Yarn NodeManager       | 1           | 1024        | allocation for the Yarn NodeManager: default 1 vcore and 1 GB |
  | Impala Daemon          | 0           | 0           | Suggestion: allocate at least 16 GB memory when using impala |
  | HBase Region server    | 0           | 0           | Suggestion: allocate no more than 12~16 GB memory when using HBase region server |

- <font color="#a00"><u>Cluster size</u></font> - enter the number of worker hosts needed to support your business case.

## 4.2 Yarn Configuration

You verify your available resources and set minimum and maximum limits for each container

Steps consist of Yarn configuration:

- <font color="#a00"><u>Verify Settings</u></font> - pulls forward the memory and vcore numbers from step [<font color="#aa0">Worker Host planning</font>]
- <font color="#a00"><u>Verify Container Settings on Cluster</u></font> 
  - <font color="#aa0">yarn.scheduler.minimum-allocation-vcores</font> - set the minimum number of vcores
  - <font color="#aa0">yarn.scheduler.maximum-allocation-vcores</font> - set the maximum number of vcores
  - <font color="#aa0">yarn.scheduler.increment-allocation-vcores</font> - set the when additional vcores are required.
  - <font color="#aa0">yarn.scheduler.minimum-allocation-mb</font> - set the minimum reservations for memory
  - <font color="#aa0">yarn.scheduler.maximum-allocation-mb</font> - set the maximum reservations for memory
  - <font color="#aa0">yarn.scheduler.increment-allocation-mb</font> - set the increment.

## 4.3 MapReduce Configuration

You can increase the memory allocation for the ApplicationMaster, map tasks, and reduce tasks. The <font color="#a00">minimum vcore</font> allocation for any task is always <font color="#a00">1</font>. The <font color="#aa0">Spill/Sort</font> memory allocation of <font color="#a00">256 (mb)</font> should be sufficient, and should be (<font color="#aa0"><u>rarely</u></font>) increased if you determine that frequent spills to disk are hurting job performance.

- <font color="#aa0">yarn.app.mapreduce.am.resource.cup-vcores</font> - AM container vcore reservation
- <font color="#aa0">yarn.app.mapreduce.am.resource.mb</font> - AM container memory reservation
- <font color="#aa0">mapreduce.map.cpu.vcores</font> - Map task vcore reservation
- <font color="#aa0">mapreduce.map.memory.mb</font> - Map task memory reservation
- <font color="#aa0">mapreduce.reduce.cpu.vcores</font> - Reduce task vcore reservation
- <font color="#aa0">mapreduce.reduce.memory.mb</font> - Reduce task memory reservation
- <font color="#aa0">mapreduce.task.io.sort.mb</font> - Spill/Sort memory reservation

