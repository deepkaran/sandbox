###Generic pub-sub

####Projector/Router

**Request New Topic**

- Index Coordinator/Indexer sends request with following information:
 - list of indexes
 - mapping between indexerid and port
 - restart timestamp

- Projector generates a new **TopicId** for this request which is for internal pub-sub use.
- TopicId is sent to router with other request payload.
- Router generates list of subscribers as follows: <br>
**TopicId, IndexerId, Port**

- Router has the topology metadata: <br>
**IndexId -> IndexInstance -> Partition/Slice -> IndexerId** 

- Projector starts a new UPR stream with KV for the bucket(s) based on timestamp for the vbuckets
it is responsible for. All messages for this connection will be sent on the newly generated **TopicId**.


####Mutation Flow on a topic

- Projector sends secondary key message to Router which contains the TopicId.
- Router will first find the partition/slice this key belongs to based on partition scheme
- Router will then figure out IndexerId this partition/slice belongs to
- Then based on the TopicId(from the message) and IndexerId, Router knows the destination for the message.

*Projector/Router do not need to know topic/stream type or the number of topics/streams*

####Destination
- The destination knows which port it has requested for what topic messages e.g. backfill, catchup, maintenance
- Even if the request has been made by the coordinator, it is a fixed port for backfill, maintenance.
But projector/router need not have that static mapping. It remains generic.
