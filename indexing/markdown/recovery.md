##Recovery Cases


##Individual Component Restart

This document describes steps for individual component restart during various states the Indexing System may be in.

####Index Coordinator(Master)


####Index Coordinator(Replica)

####Projector/Router

Projector and Router are stateless components and run in a single process on KV.
These components don't have a local persisted state. 

**Process Crash**

**Node Restart**


####Indexer (Local Indexer)







| Failure       | Mode         | Topic  | Action  |
| ------------- |:-------------:| -----:| -----:|
| UPR Connection Loss      | Regular | Maintenance |  |
| UPR Connection Loss      | Regular | Catchup |  |
| UPR Connection Loss      | Regular | Backfill |  |
| UPR Connection Loss      | Rollback | Maintenance |  |
| UPR Connection Loss      | Rollback | Catchup |  |
| UPR Connection Loss      | Rollback | Backfill |  |
| Projector Restart      | Regular      | Maintenance |  |
| Projector Restart      | Regular      | Catchup |  |
| Projector Restart      | Regular      | Backfill |  |



Open Questions

- Do we keep a connection open from IC to projector?
- If we piggyback SYNC messages, then who sends it to projector
- Does IC contact all projectors for the log in case of one projector dropping UPR connection
