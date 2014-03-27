###Proposal for generic pub-sub

####Projector

**Request New Topic**

- Request contains:
 - list of indexes
 - mapping between indexerid and port

Projector generates a new topicid for this request which is for internal pub-sub use.
TopicId is sent to router with other request payload.

####Router

Router has the topology metadata:
<IndexId>
-- <IndexInstance> (For replica)
---- <Partition>
------ <Slice>
-------- <IndexerId>

Router generates list of subscribers as follows:
<TopicId><IndexerId><Port>


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

####Constraints
- Make sure projector can open multiple connection request(for multiple buckets) for the same topic
- Works for topics which need to be sent to all Indexers vs subset of Indexers
- Multiple destinations for a single topic
