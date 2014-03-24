##Design Writeup Tasks

- Metadata Management(write briefly about sync replication and then point to Initial build prepare phase)
- Initial Build Writeup(Add Catchup Queue Stuff)
- Gerrit Reviews update
- Review this doc and add recovery questions
- Review all writeups
- Indexer writeup
 - Include a separate port for indexer
- Writeup for Drop Index


##Other Tasks

- Understand 2PC
- Understand Raft
- John's doc on snapshot requirement
- Reply to mails (John's arch ppt, Long mail)

##Questions

- Is our UPR assumption correct that there is no data loss if there is no failover log entry
e.g. restart of memcached

- How do we query from replica if index coordinator is not going to participate

- how does index client get notifications about topology changes, new ddl requests from other clients

- Do we create separate endpoints for each mutation queue/catchup queue?

####Recovery

- Think about recovery with multiple indexes being at different points

##Convergance

####Stability Timestamp Promotion

- If coordinator does not poll, how will indexer know where its peers are? How does it choose Scan Timestamp? Will it talk to other indexers at scan time or will it be told about other indexers status e.g. replica may have caught up and indexer would like to know that
- If it polls, maybe still it can keep generating ST in future and doesn't wait 
- May be its a question of allowing more control per index, if we leave this decision to indexer at scan time. coordinator doing it would make it per bucket sort of.

####Initial Build

- Multiple Catch queue - Is indexer allowed to have multiple catchup queues for maintenance/catchup?

####Execution Flow

- Index Update

  As we don't wait for replica to catchup, before promoting stability timestamp
how will indexer know if it can issue scan request for the replica

- Step 5

  The local indexer checks if its high watermark timestamp is higher than the stability timestamp.  
If not, then it will return to the index manager with its high watermark timestamp.  
How is this possible, we have a already done this is Step 2,3


####Recovery
- What if IC dies before all indexers have persisted and one of the indexer also dies?

- Why is Restart Timestamp -> stability snapshot

- What if all indexers couldn't listen to stability ts message before IC died

- What happens during the duration there is no master


- In case of rollback/restart, catchup has to be discarded as we supplied the lowest timestamp
so everybody would have rolled-back to same state, and no catch-up is required.

####General

- Mechanism of catchup in case of multiple indexes(from multiple buckets) or for initial load in the same indexer node, specifically how indexer will differentiate which message is for which topic?

- Local Indexer will persist mutations from catchup queue based on Stability Timestamp history. 
It will also create snapshots based on Stability Timestamp history. 

- Projector failure when catchup is happening

- Why is snapshot creation responsibility of the local indexer i.e. the choice of when to create

- How does new master check everything is running fine


##Box
Main flows in the system:

- Insert/Update Mutation
- Delete Mutation
- Initial Build
- Scan Query
- Stability Timestamp Details
- Recovery
--Persistent Snapshots
--KV Rollback
--Rollback
--Local Indexer Restart

- Bootstap Flows(initial, restart)
- Pub-Sub
- Metadata Management

