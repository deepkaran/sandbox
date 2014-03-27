##Design Writeup Tasks

####Today

- update query.md to use coordinator terminology(arrow in step13 is missing)
- link overview.md to README
- Workout topic stuff
- Initial Build Writeup(Add Catchup Queue Stuff, Also mention new mutations will be queued up in mutation queue)
 - Step 10,  indexing building -> initial load or backfill
 - status INITIAL_BUILD should be marked by coordinator?
 - make build complete a proper message

####Later
- Add review comment for invariant
- Read snapshot document again
- Catchup Queue working needs a flow diagram
- Recovery needs a flow diagram
- Understand KV Rebalance handling in index
- Index Coordinator restart
- Review JIRA Tasks

##Pending Tasks
- Indexer writeup
 - Include a separate port for indexer
- Writeup for Drop Index
- Recovery Flow diagram

##Edits

##Other Tasks

- Understand 2PC
- Understand Raft
- John's doc on snapshot requirement
- Reply to mails (John's arch ppt, Long mail)

##Questions

- Is our UPR assumption correct that there is no data loss if there is no failover log entry
e.g. restart of memcached
- how does index client get notifications about topology changes, new ddl requests from other clients

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
- When do we start persisting as per stability timestamps?
 - When init build is in progress, indexer keeps storing history and then apply after build is complete
 - Or after build is complete, then it stores and apply 

####Recovery

- Step 7
If the failover timestamp is lower than the UPR restart timestamp, 
then the local indexer will find a snapshot that is higher than or equal to the UPR restart timestamp.
How is the above possible.

####Index Coordinator restart

- When replica takes over, it needs to check if it is rollback mode
- What if all indexers couldn't listen to stability ts message before IC died
- What happens during the duration there is no master
- What is the procedure for verification when coordinator comes up?

####General

- Local Indexer will persist mutations from catchup queue based on Stability Timestamp history. 
It will also create snapshots based on Stability Timestamp history. 
- Who verifies the topology once indexer comes up, index coordinator or indexer?
- Why does router drop the slow subscriber? It still has to get the mutations even if it establishes a catchup. I assume that main connection also get re-activated once catch-up has been requested.
- No component restarts itself, only ns_server can bring up components. So effectively bootstrap and restart are the same.

**Important**
- Backfill queue won't require rollback if new mutations are put in mutation queue only
- Backfill queue will be done after reaching initial build timestamp
- Recovery: Catchup queue will be done once we have reached the point where recovery started and things are in mutation queue. (Do we apply stability timestamp to catchup queue?)


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

