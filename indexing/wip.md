##Design Writeup Tasks

- Review Pratap's Docs
- Partition/Slice mapping
- Send mail for differences in recovery doc
- Add to bootstrap that indexer will verify its local copy
- Initial Build Writeup(Add Catchup Queue Stuff)
- Add Readme file for review
- Indexer writeup
 - Include a separate port for indexer
- Writeup for Drop Index
- Review JIRA Tasks
- Review all writeups

##Other Tasks

- Understand 2PC
- Understand Raft
- John's doc on snapshot requirement
- Reply to mails (John's arch ppt, Long mail)

##Questions

- Is our UPR assumption correct that there is no data loss if there is no failover log entry
e.g. restart of memcached
- how does index client get notifications about topology changes, new ddl requests from other clients
- Does projector talk to IC to register itself?


##Convergance

####Stability Timestamp Promotion

- If coordinator does not poll, how will indexer know where its peers are? How does it choose Scan Timestamp? Will it talk to other indexers at scan time or will it be told about other indexers status e.g. replica may have caught up and indexer would like to know that
- If it polls, maybe still it can keep generating ST in future and doesn't wait 
- May be its a question of allowing more control per index, if we leave this decision to indexer at scan time. coordinator doing it would make it per bucket sort of.
- How do we query from replica if index coordinator is not going to participate


####Initial Build

- Multiple Catch queue - Is indexer allowed to have multiple catchup queues for maintenance/backfill?
How does indexer differentiate between messages for same catchup queue, is it based on different endpoint?
- Do we create separate endpoints for each mutation queue/catchup queue?
- What if after initial build is complete, backfill queue is being used and user wants to start another round of build procedure? Is it better to call index is ready only when it has caught up completely?

####Execution Flow

- Index Update

  As we don't wait for replica to catchup, before promoting stability timestamp
how will indexer know if it can issue scan request for the replica

- Step 5

  The local indexer checks if its high watermark timestamp is higher than the stability timestamp.  
If not, then it will return to the index manager with its high watermark timestamp.  
How is this possible, we have a already done this is Step 2,3


####Recovery

- In case of rollback/restart, catchup has to be discarded as we supplied the lowest timestamp
so everybody would have rolled-back to same state, and no catch-up is required.

- Why we clean up all queues if in rollback mode, 
queues may be full as indexers are slow, but this data can be useful

- Step 7
If the failover timestamp is lower than the UPR restart timestamp, 
then the local indexer will find a snapshot that is higher than or equal to the UPR restart timestamp.

How is the above possible.

- Step8
How can we assume that backfill mutations would never need to be rolled back

- When does maintenance stream start in case of KV rollback


####Index Coordinator restart

- When replica takes over, it needs to check if it is rollback mode
- What if all indexers couldn't listen to stability ts message before IC died
- What happens during the duration there is no master

####General

- Local Indexer will persist mutations from catchup queue based on Stability Timestamp history. 
It will also create snapshots based on Stability Timestamp history. 


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

