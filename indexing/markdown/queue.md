##Queues

####Type

- Mutation Queue
- Catch Queue
- Backfill Queue


_For each type how many internal queues are required:_

Variables:
- vbucket
- bucket
- index
- size of mutation

Prototype has 8 internal worker queues for each main Queue.
This improves the persistence rate due to parallel workers.

####Execution Flow:

- Do Sanity e.g. if slice doesn't exist on this node, return NOT\_MY\_SLICE, match the vbuuid 
- Check for the Delete Marker
- If there is a open snapshot, apply mutation to forestdb but don't commit
- Else Put the mutation on the MutationQ based on bucketid
 - Put mutation on a worker queue based on vbucketid
 - Worker will persist the mutation(update main/back index)
- Create snapshot and store the timestamp in separate persisted file
- Update HWT for the queue


####Memory Management:

- Fixed Size Queues


- Dynamic Queues 

  As physical machines can be varied in compute power, dynamic queue(say with max 30% memory) is a better option.
  Queue should be able to store lot of mutation as persistence is expected to be slow.
  
####System Constraints

- Need to be able to throttle serving queries when system is running low on memory.
- Need to be able to throttle reading mutations off the network if system is running low on memory. This will inturn trigger throttle at the router end.
- New index can be added any time which will require another copy of MutationQ.
- Mutation queue will have mutation with same seqno/vbucketid for multiple indexes and that needs to be accounted while taking any decisions.


####Implementation

__Global Pool__
- Global Pool can be maintained per queue or per queue type(e.g. mutation, catchup, backfill)
- Pool is a large array of Mutation objects
- Each struct needs to have a flag which says its free (Or maybe its a array of structs which have a free flag)
- Needs to have a free list as well, so its not required to traverse the full list to find the free slot
- When a mutation message is read its in the protobuf struct, and then if the list has a free slot its copied. Otherwise the thread stalls to do it, thereby causing underlying library to stall reading from the network.

__Mutation Queue__
- Circular buffer (array of say 1M mutation pointers)
- Head and Tail are maintained
- If there is no open stability timestamp, mutations are put in MutationQ
- Each pointer also has a `USED` flag, which can be used to flag the fact that mutation entry has been processed and the place is reusable.
- It is expected that lower seqno mutations are ahead in the queue and will get processed first. So fragmentation should be taken care of automatically.
- As there can be multiple entries for a single seqno/vbucket combination(due to multiple indexes), the persistence processing stops when it
 - finds a seqno higher for the vbucket
 - reaches the end of queue

__Flow__

- Indexer is running a tcp server
- Router can create multiple connections to this port but it will use one for every vbucket
- At server, all the mutations coming in are put on a single channel say ch1
- Goroutine listening to ch1 will just pick the mutation and put them on a set of channels lets say ch1a to ch1d based on vbucket
- Goroutine listening to ch1a to ch1d will copy the protobuf message on to mutation struct allocated by SlabManager which encapsulates go-slab (making it concurrent and also limiting the max size)
- Then this mutation message is added on to the queue
- Queue will internally increment its HWT and head/tail pointer for this vbucket. 


- *How will flow control happen*
- *Thread Safety needs to be built*
- *Optimize for open ST i.e. bypass the queue and onto the persistence directly*
- *Add function for draining the queue*
