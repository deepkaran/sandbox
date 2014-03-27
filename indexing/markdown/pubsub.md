###Generic pub-sub

####Projector/Router

**Request New Topic**

- **Index Coordinator/Indexer** sends request with following information:
 - List of IndexIds
 - `Subscriber List` which is a list of mapping between IndexerId(each indexer has a unique id) and Port(on which Indexer wants to receive the message)
 - Restart timestamp

*`Subscriber List` would have list of mapping for all Indexers in case of Maintenance/Backfill and only 1 for Catchup. This design is generic and a subset of indexers can be provided as well.*

- Projector generates a new **TopicId** for this request which is for internal pub-sub use.
- TopicId is sent to router with `Subscriber List`.
- Router stores `Subscriber List` as follows: <br>
**TopicId, IndexerId, Port**

- Router has the topology metadata: <br>
**IndexId -> IndexInstance -> Partition/Slice -> IndexerId** 

- Projector starts a new UPR connection with KV for the bucket(s) based on timestamp for the vbuckets
it is responsible for. All messages for this UPR connection will be sent on the newly generated **TopicId**.


####Mutation Flow on a topic

- Projector sends secondary key message to Router which contains the TopicId.
- Router will first find the partition/slice this key belongs to based on partition scheme
- Router will then figure out IndexerId this partition/slice belongs to
- Then based on the TopicId(from the message) and IndexerId, Router looks up the `Subscriber List` to find which Port on the destination the message needs to be sent.


*Projector/Router do not need to know __TYPE__ of stream/topic.*

####Destination
- The destination knows which port it has requested for what topic messages e.g. backfill, catchup, maintenance
- Even if the request has been made by the coordinator, it is a fixed port for backfill, maintenance.
But projector/router need not have that static mapping. It remains generic.
