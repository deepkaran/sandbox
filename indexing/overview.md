#Design Overview for Indexing


##Overview

This document describes the High Level Design for Secondary Indexes. It also describes the deployement options supported.

##Components


- __Projector__

  The projector is responsible for mapping mutations to a set of key version <key, docId, vbucketId, seqNo>.    The projector can reside within the master KV node in which the mutation is generated or it can reside in separate node. The projector receives mutations from ep-engine through UPR protocol. The projector sends the evaluated results to router.

- __Router__

  The router is responsible for sending key version to the index nodes. The router resides in the same node as the projector.
  
- __Index Manager__

  The index manager is responsible for receiving requests for indexing operations (creation, deletion, maintenance, scan/lookup). The Index Manager is located in the index node, which can be different from KV node. 
  
- __Indexer__

  The indexer provides persistence support for the index. The indexer would reside in index node. 
  
- __Query Catalog__

  This component provides catalog implementation for the Query Server. This component resides in the same node Query Server is running and allows Query Server to perform Index DDL (Create, Drop) and Index Scan/Stats operations.



##System Diagram
![KV And Index Cluster](https://rawgithub.com/deepkaran/sandbox/master/indexing/images/SystemDiagram.svg)

##Deployement Diagram

##Mutation Workflow


##Query Workflow


##Storage Management


##Cluster Management


##Metadata Management


##Recovery
[Recovery Document](https://docs.google.com/document/d/1rNJSVs80TtvY0gpoebsBwzhqWRBJnieSuLTnxuDzUTQ/edit) 

##Replication


##Rebalance


##Terminology

- High-Watermark Timestamp
- Stability Timestamp
- Restart Timestamp
- Mutation Queue
- Catchup Queue
- Stability Snapshot
- Persistent Snapshot
