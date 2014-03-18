##Execution flow of DDL 


![](https://rawgithub.com/deepkaran/sandbox/master/indexing/images/DDLWorkflow.svg)

####Description

#####Prepare Phase

1. The Index Coordinator receives CREATE INDEX request from client.
2. A unique index id is generated, the state of index is marked as INIT, StateContext is updated with proposed topology information and CAS is incremented for StateContext.
3. StateContext is persisted locally so it can be recovered if Index Coordinator restarts.
4. StateContext is replicated to *ALL* Index Coordinator Replicas synchronously i.e. Index coordinator waits for each replica to ACK the replication request.
  -  If Replica doesn't reply with ACK, Index Coordinatr will retry
  -  If retry doesn't succeed, Coordinator can check with ns_server if replica is down.
5. Index Coordinator Replica will locally persist the StateContext and send ACK back to the master. 
  - If Master dies and this replica gets promoted to master, it can resume the index building or do cleanup depending on the state of the system.
  - The new master must consolidate the StateContext from other replicas. It will pick the highest version/CAS of the StateContext among the replica. The new master is responsible for synchronize the state again among the replica.
6. Same as Step 4(Step 4 and 5 will be repeated for all index coordinator replicas).
7. Same as Step 5.
8. The Index Coordinator sends Create Index request to *ALL* Indexers which are part of index topology. 
9. Local Indexer will allocate storage for the new index, update its local metadata and sends ACK to coordinator.
10. Same as Step 8(Step 8 and 9 will be repeated for all indexers hosting this index).
11. Same as Step 9.
12. Once Index Coordinator has received ACKs from all Indexers, Index State will be marked as READY, StateContext will be updated and CAS gets incremented. The new StateContext is persisted locally.
13. Index Coordinator will replicate new StateContext to *ALL* Index Coordinator Replicas synchronously.
14. Index Coordinator Replica will locally perist the StateContext and send ACK back to the master.
15. Same as Step 13(Step 13 and 14 will be repeated for all index coordinator replicas).
16. Same as Step 14.
17. Index Coordinator returns SUCCESS to client.


*After Prepare phase, it will require the administrator to initiate index building from UI. <br>
The administrator will have to select the new index definitions for initial loading.*

#####Load Phase

18. Index coordinator will notify all Routers about new index topology. 
19. Router will create topics/subscribers for the new index and ACK to index coordinator.
20. Index Coordinator will notify all Projectors about the new index definitions. 
21. Projector will star

Notes:

- If there is network partitioning before the replication is completed,
the algorithm would allow the master to continue to replicate the
StateContext to the other replica once the network is healed.  This is
expecting that there is no split brain (automatic failover will be off if
ns_server detects more than one node is down).  There is still a corner
case that split brain may still happen (the master looses connectivity).
But this algorithm should work in this case since there is at least one
replica has the replicated StateContext.

- Let's say the master has replicated the state to a set of replica and
then the rack fails.  Let's also say that for some strange reason, the
master and those replica are in the same rack.  In this case if a
new master is selected, it will not see the new StateContext.   In this
case, we have to consider this request to fail.  When the rack rejoins,
those nodes will have to restart and they have to sync up with the
StateContext from the new master.
