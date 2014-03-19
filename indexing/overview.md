##Secondary Index Design Document


###Overview

This document describes the High Level Design for Secondary Indexes. It also describes the deployement options supported.

###Components


- __Projector__

  The projector is responsible for mapping mutations to a set of key version. The projector can reside within the master KV node in which the mutation is generated or it can reside in separate node. The projector receives mutations from ep-engine through UPR protocol. The projector sends the evaluated results to router.

- __Router__

  The router is responsible for sending key version to the index nodes. It relies on the index distribution/partitioning topology to determine the indexer which should receive the key version. The router resides in the same node as the projector.
  
- __Index Manager__

  The index manager is responsible for receiving requests for indexing operations (creation, deletion, maintenance, scan/lookup). The Index Manager is located in the index node, which can be different from KV node.
  
- __Indexer__

  The indexer provides persistence support for the index. The indexer would reside in index node.
  
- __Query Catalog__

  This component provides catalog implementation for the Query Server. This component resides in the same node Query Server is running and allows Query Server to perform Index DDL (Create, Drop) and Index Scan/Stats operations.


###System Diagram

- [KV-Index System Diagram](markdown/system.md)
- [Query-Index System Diagram](markdown/system_query.md)

###Execution Flow

* [Initial Index Build](markdown/initialbuild.md)
* [Mutation Execution Flow](markdown/mutation.md)
* [Query Execution Flow](markdown/query.md)

###Bootstrap Sequence

* System Bootstrap
* Indexer Restart Bootstrap
* Projector Restart Bootstrap
* Router Restart Bootstrap
* Index Manager Restart Bootstrap

###Deployment

- [Deployment Options](markdown/deployment.md)

###Partition Management
* Milestone1 will have Key-based partitioning support. 
  * [John's Doc for Partitioning](https://docs.google.com/document/d/1eF3rJ63iv1awnfLkAQLmVmILBdgD4Vzc0IsCpTxmXgY/edit)

###Communication Protocols

* Projector and Ep-Engine Protocol
  * [UPR protocol](https://github.com/couchbaselabs/cbupr/blob/master/index.md) will be used to talk to Ep-engine in KV. 

* Router and Indexer Protocol
* Query and Indexer Protocol
  * [Existing REST Based Protocol](https://docs.google.com/document/d/1j9D4ryOi1d5CNY5EkoRuU_fc5Q3i_QwIs3zU9uObbJY/edit)

###Storage Management
* Persistent Snapshot 

###Cluster Management
* Master Election
* Communication with ns_server

###Metadata Management
* Metadata Replication
* Metadata Recovery

###Recovery
* [John's Recovery Document](https://docs.google.com/document/d/1rNJSVs80TtvY0gpoebsBwzhqWRBJnieSuLTnxuDzUTQ/edit) 

###Replication
* Replication Strategy
* Failure Recovery

###Rebalance
* Rebalance Strategy
* Failure Recovery

###Terminology

* [Terminology Document](markdown/terminology.md)

###References
1. [Distributed Indexing Design Proposal - Steve Yen](https://docs.google.com/document/d/1TEY_yjUMs3FT3FZkgqIZUiKziUUBGiCWFSjWOOj65Dw/edit?pli=1)
2. [Distributed Range Partitioned Indexes – Steve Yen, Dustin Stallings](https://docs.google.com/presentation/d/161vtnjDFOpnliT0DA2k1-jCRKiB-eY9JZHhd0YQoAXo/edit#slide=id.p)
3. [Index Consistency And Isolation - Steve Yen](https://docs.google.com/document/d/1Y_aXMUBzEvLf8PO8CJYv5eYiQmKsNYzMr6Fq30Cl6xg/edit?pli=1)
4. [Recovery In Secondary Indexes - John Liang](https://docs.google.com/document/d/1rNJSVs80TtvY0gpoebsBwzhqWRBJnieSuLTnxuDzUTQ/edit) 
5. [Consistency In Indexes - John Liang](https://docs.google.com/document/d/1VespzgCKgPLFwCGRrx0VW7RC_PkSXh6cU7FSaA9gjic/edit?pli=1)
6. [Durable Transactions - John Liang](https://docs.google.com/document/d/11Rm2fW9Punx4tktUkK5Re9P87N8_aUwaEJw3u7XR_UY/edit?pli=1)
