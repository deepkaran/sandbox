##Couchbase Secondary Indexes

Couchbase Secondary Indexes are being designed to provide an alternative to the existing "Views based" secondary indexes available. This project is being built from ground up to improve existing solution and support new use-cases e.g. N1QL.

###Goals and Motivation
- Independent Scaling – Ease of scaling by allocating dedicated resources for different kinds of workload (e.g. KV versus indexing)
- Enable predictable index range scan latency (e.g. Independent of KV cluster size)
- Improve latency and throughput on existing solution
- Support N1QL use case
  - Query Stability 
  - Consistency Requirement

###Design Principle
- Timestamp based consistency/stability
- Event Streaming and Routing – E.g. An index replica can also be a source of event generation (not just projector)
- Network Protocol Independent
- Active Replica (Allow query on active replica to enable higher throughput)
- Partitioning Scheme Independent 
- Storage Layer Independent 

###Version1 Features

#####Indexing
- Key Based Partitioning Support
- Cluster Manager using ns_server(master election)
- ForestDB Integration as backend for persistence/query
  - Support for Crash Recovery and Compaction
- Index Node Failover and Rebalance
- Distributed Index Metadata Management
- Error Management (Recovery for all Indexing component failures)
- Administration UI (Management And Statistics)
- Network Protocol Independent
- Flexible Deployment Options

#####KV Related
- Independent Scaling from KV Cluster 
- Mutation Stream via UPR
- Support KV Failover(data loss), Rebalance

#####Query Related
- Consistency/Stability Options
- Active Replica for Query

#####Limitations


