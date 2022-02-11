# Spanner

## Paper

[Spanner: Google’s Globally-Distributed Database](https://static.googleusercontent.com/media/research.google.com/en//archive/spanner-osdi2012.pdf)

## Short description

Spanner is a scalable, multi-version database that offers wide-area synchronous replication and distributed transactions.

## To expand later:

- read-heavy focus
- strong consistency

- each datacenter has shards of all data
- each datacenter has local clients
- one Paxos (log-based with leader) group per shard 

- datacenters fail independently
- shards allow higher throughput with parallelism
- clients can read locally
- Paxos can tolerate some failures/slowdowns, only needs majority

- r/w transactions with 2PC (but TC is replicated so no blocking)
- client coordinates transaction, chooses a Paxos group as TC

- r/o transactions without locking

- Snapshot Isolation
- TrueTime
