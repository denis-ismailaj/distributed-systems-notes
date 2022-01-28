# MapReduce

## Paper

[MapReduce: Simplified Data Processing on Large Clusters](https://research.google/pubs/pub62.pdf)

## Short description

MapReduce is a programming model and an associated implementation for processing and generating large data sets.

## Usage

The computation takes a set of _input_ key/value pairs, and produces a set of _output_ key/value pairs.
The user of the MapReduce library expresses the computation as two functions: _Map_ and _Reduce_.

_Map_, written by the user, takes an input pair and produces a set of _intermediate_ key/value pairs.
The MapReduce library groups together all intermediate values associated with the same intermediate key _I_
and passes them to the _Reduce_ function.

The _Reduce_ function, also written by the user, accepts an intermediate key _I_ and a set of values for that key.
It merges together these values to form a possibly smaller set of values.

Programs written in this functional style are automatically parallelized and executed on a large cluster of machines. 
The run-time system takes care of the details of partitioning the input data, 
scheduling the program's execution across a set of machines,
handling machine failures, and managing the required inter-machine communication.

This allows programmers without any experience with parallel and distributed systems
to easily utilize the resources of a large distributed system.

### Example

The problem of counting the number of occurrences of each word in a large collection of documents
could be implemented in MapReduce with code similar to the following pseudo-code:

    map(String key, String value):
        // key: document name
        // value: document contents
        for each word w in value:
            EmitIntermediate(w, "1");

    reduce(String key, Iterator values):
        // key: a word
        // values: a list of counts
        int result = 0;
        for each v in values:
            result += ParseInt(v);
        Emit(AsString(result));

## Execution Overview

_Map_ invocations are distributed across multiple machines by automatically partitioning the input data
into a set of _M_ splits.

_Reduce_ invocations are distributed by partitioning the intermediate key space into _R_ pieces 
using a partitioning function (e.g. _hash(key) **mod** R_). 
The number of partitions (_R_) and the partitioning function are specified by the user.

![_Figure 1_ from the paper](https://i.imgur.com/BvFOTJj.png)

1. The MapReduce library first partitions the input data.
It then launches many instances of the program spread throughout the cluster.

2. One of the instances serves as the master and the rest as workers.
The master picks idle workers and assigns each one a task.
There are _M_ map tasks and _R_ reduce tasks to assign.

3. A worker who is assigned a map task reads the contents of the corresponding input split.
It parses key/value pairs out of the input data and passes each pair to the user-defined _Map_ function.

4. The intermediate key/value pairs produced by the _Map_ function are written to local disk, 
partitioned into _R_ regions by the partitioning function.
The locations of these pairs on the local disk are passed back to the master, who is responsible for forwarding
these locations to the reduce workers.

5. When a reduce worker is notified by the master about the locations for the region it was assigned, 
it uses remote procedure calls to read the data from the local disks of the map workers.
When it has read all intermediate data, it sorts it by the intermediate keys so that all occurrences 
of the same key are grouped together. 
The sorting is needed because typically many different keys map to the same reduce task.

6. The reduce worker iterates over the sorted intermediate data and for each unique intermediate key encountered, 
it passes the key and the corresponding set of intermediate values to the user's _Reduce_ function. 
The output of the _Reduce_ function is appended to a final output file for this reduce partition.

7. When all map tasks and reduce tasks have been completed, the master notifies the user and the MapReduce execution terminates.

After successful completion, the output of the MapReduce execution is available in the _R_ output files (one per reduce task).

Typically, users pass these files as input to another MapReduce call, 
or use them from another application that can handle partitioned input.

## Fault tolerance

### Worker failure

- Worker failure is detected via periodic heartbeats from the master.
- All previously completed map tasks by the worker are rescheduled (because their output is only stored locally).
- Similarly, any map or reduce task in progress on a failed worker is also rescheduled.
- The master ignores duplicate completion reports for the same task.

### Master failure

Given that there is only a single master, its failure is unlikely.
However, failure recovery could be implemented using periodic checkpoints of the master's data,
so that a new instance can take over using the checkpointed state.

## Semantics

When the user-supplied _map_ and _reduce_ operators are deterministic functions of their input values,
MapReduce produces the same output as would have been produced by a non-faulting sequential execution
of the entire program.

## Optimizations

- When using a distributed file system, the MapReduce master can make locality-aware decisions 
to schedule tasks on machines that already have a replica of the task's input data or that are near a replica.
This way, most input data can be read locally and consumes no network bandwidth.

- When a MapReduce operation is close to completion, the master schedules backup executions
of the remaining in-progress tasks in order to prevent laggards from delaying the entire operation.

- When the user's _Reduce_ function is commutative and associative, an optional _Combiner_ function 
can be defined that partially _reduces_ the map outputs locally before the data is sent over the network.

- When workers crash deterministically on certain records, MapReduce can be configured
to detect and skip these records.
