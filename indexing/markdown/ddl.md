##Execution flow of DDL 


![](https://rawgithub.com/deepkaran/sandbox/master/indexing/images/DDLWorkflow.svg)

Corner Cases:

- If a replica does not reply, the index coordinator can retry to ensure
state is replicated.   The master can confirm if a replica is down through
ns_server (no further retry needed).

- If the master dies, a new master is elected.  The new master must
consolidate the StateContext from other replica.    It will pick the
highest version/CAS of the StateContext among the replica.   The new
master is responsible for synchronize the state again among the replica.

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
master and those replica are in the same rack.  In this case, I am not
sure if there will be a new master since multiple node fails.   But if a
new master is selected, it will not see the new StateContext.   In this
case, we have to consider this request to fail.  When the rack rejoins,
those nodes will have to restart and they have to sync up with the
StateContext from the new master.
