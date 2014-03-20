##System Bootstrap Sequence


System bootstrap will be initiated by ns_server. It will bring up all the processes with required parameters.
Once a process is up, it follows its own bootstrap sequence and then either waits for further commands to be given 
or contacts Index Coordinator to get the latest StateContext and takes further actions based on that.

ns_server also watchdogs each process and restarts it in case of failure/crash.

In case a node goes down, ns_server will detect that and pass this information to Index Coordinator Master.
If Index Coordinator Master node has gone down, ns_server will elect a new master.

####Brief Steps

1. ns_server brings up Index Coordinator Master.
2. ns_server brings up Index Coordinator Replicas providing master endpoint as input. All replicas handshake with master to sync up.
3. Index Coordinator chooses the latest StateContext among its local copy and all the replicas.
4. Index Coordinator master sends ACK to ns_server.
5. ns_server brings up Projector/Router on all KV nodes. Projector and Router are stateless components. Both the components listen on their admin ports for messages from other indexing components.
6. ns_server provides the list of Projectors/Routers to Index Coordinator master.
7. Index Coordinator master updates the local StateContext with this information and persist it.
8. State Context is replicated to all Index Coordinator Replicas.
9. Index Coordinator master sends ACK to ns_server.
10. ns_server brings up Indexer process on all Indexer Nodes providing master endpoint as input.
11. Indexer process will handshake with Index Coordinator master and get the StateContext (which has index topology and list of projectors). 
12. Once all Indexers have done the handshake, Index Coordinator will send request to all Projectors to initiate `Topic Creation` for `IncrementalStream` topic sending StateContext to Projector.
13. One or more Indexers may choose to create `Catchup Stream` based on if its in recovery.




####Description

1. ns\_server will have a list of `Indexer Nodes` as part of its config. This list gets updated if nodes are added/removed from the Admin UI. When the system starts, ns\_server will designate one node as master and spawn an instance of Index Manager(called Index Coordinator from this point) with Master role. Index Coordinator Master will check if it has a locally persisted StateContext.

 - If this is a fresh election for this master, it will not find any local StateContext. It waits for replicas to register with it.
 - If this is a process restart(crash/node restart), it will find a local StateContext. It then waits for replicas to register with it.
 - If this is a promotion from replica to master, local StateContext will be found. New master needs to make sure it has the latest replicated StateContext. Again it waits for replicas to register with it. 

2. ns_server will further designate a set of nodes(based on config param) as replicas and spawn an instance of Index Manager(Index Coordinator) with Replica role providing the endpoint of Master.
 - It this is a fresh election for this replica, it will not find a local StateContext. It will register with master, asking for StateContext.
 - It this is a process restart(crash, node restart), it will find a local StateContext. It will register with master, providing the version number of its StateContext. 
 - If ns_server elects a new master, it will inform all existing replicas with the new endpoint. All replicas need to register with the new master to sync up.

3. Once all replicas have registered with Index Coordinator master, it will compare the local StateContext version with the version received from replicas and choose the highest version of StateContext. If it doesn't have the highest version, it will ask the replica to provide the StateContext. 
 - If this was a fresh master election or promotion from replica to master, ns_server will inform all indexers with the new master endpoint.
 - If this was a process restart, indexers will automatically retry and re-establish the connection.

4. Index Coordinator master will send ACK to ns_server to proceed with rest of system bootstrap.
5. ns_server brings up Projector/Router on all KV nodes. Projector and Router are stateless components. Both the components listen on their admin ports for messages from other indexing components.
 - If this is a indexing system restart, ns_server doesn't need to restart projector/router.
6. ns_server provides the list of Projectors/Routers to Index Coordinator master.
7. Index Coordinator master updates the local StateContext with this information and persist it.
8. State Context is replicated to all Index Coordinator Replicas.
9. Index Coordinator master sends ACK to ns_server.
10. ns_server brings up Indexer process on all Indexer Nodes providing master endpoint as input.
11. Indexer process will handshake with Index Coordinator master and get the StateContext (which has index topology and list of projectors). 
12. Once all Indexers have done the handshake, Index Coordinator will send request to all Projectors to initiate `Topic Creation` for `IncrementalStream` topic sending StateContext to Projector.
13. One or more Indexers may choose to create `Catchup Stream` based on if its in recovery.

