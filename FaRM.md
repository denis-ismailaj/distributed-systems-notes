# FaRM

## Paper

[No compromises: distributed transactions with consistency, availability, and performance](https://dl.acm.org/doi/pdf/10.1145/2815400.2815425)

## Short description

FaRM is a main memory distributed computing platform that provides distributed transactions with strict serializability, high performance, durability, and high availability.

## Unnamed

FaRM uses one-sided RDMA operations where possible because they do not use the remote CPU. 

FaRM provides applications with the abstraction of a global address space that spans machines in a cluster. 
Each machine runs application threads and stores objects in the address space. 
The FaRM API [16] provides transparent access to local and remote objects within transactions. 
An application thread can start a transaction at any time and it becomes the transaction’s coordinator. 
During a transaction’s execution, the thread can execute arbitrary logic as well as read, write, allocate, and free objects. 
At the end of the execution, the thread invokes FaRM to commit the transaction.
FaRM transactions use optimistic concurrency control.

FaRM provides strict serializability [35] of all successfully committed transactions. 
During transaction execution, FaRM guarantees that individual object reads are atomic, 
that they read only committed data, that successive reads of the same object return the same data, 
and that reads of objects written by the transaction return the latest value written. 
It does not guarantee atomicity across reads of dif- ferent objects but, in this case, 
it guarantees that the transaction does not commit ensuring committed transactions are strictly serializable. 

The FaRM API also provides lock-free reads, which are optimized single-object read only transactions, and locality hints, which enable programmers to co-locate related objects on the same set of machines

FaRM uses a Zookeeper [21] coordination service to ensure machines agree on the cur- rent configuration and to store it, as in Vertical Paxos [25]. 
But it does not rely on Zookeeper to manage leases, detect failures, or coordinate recovery, as is usually done. 
The CM does these using an efficient implementation that leverages RDMA to recover fast. 
Zookeeper is invoked by the CM once per configuration change to update the configuration.

FaRM uses leases [18] to detect failures. 
Every machine (other than the CM) holds a lease at the CM and the CM holds a lease at every other machine. 
Expiry of any lease triggers failure recovery. 
Leases are granted using a 3-way handshake. 
Each machine sends a lease request to the CM and it responds with a message that acts as both a lease grant to the machine and a lease request from the CM. 
Then, the machine replies with a lease grant to the CM.
