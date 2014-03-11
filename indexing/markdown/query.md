##Query Execution Flow

This document describes the flow of execution of a query engine request.

####Create/Drop DDL Request

####Scan Request 

![](https://rawgithub.com/deepkaran/sandbox/master/indexing/images/ScanWorkflow.svg)

**Description**

1. Index Client(query catalog implementation which resides on query server) receives index scan request from the Query Server Component. 
2. Index Client will choose a Indexer node to send this request to. Index Client will have a list of available indexer nodes.
3. Index Client will choose(or will be provided by query engine) a Consistency/Stability option for the index scan. This is based on the consistency/latency requirements of the application issuing the query.

  __Consistency Options__
  - Any Consistency : The indexer would return the most current data available at the moment.  
  - Session Consistency : The indexer would query the latest timestamp from each KV node.   It will ensure that the scan   result is at least as recent as the KV timestamp.  In other words, this option ensures the query result is at least as recent as what the user session has observed so far.   

  __Stability Options__
  - No Stability.   The indexer does not ensure any stability of data within each scan.  This is the only stability option for consistency-Any.
  - Scan Stability : The indexer would ensure stability of data within each scan.
  - Query Stability.  The indexer would ensure stability of data between multiple scans within a query.

4. Scan Request is sent to the Indexer identified in Step 2 along with the consistency/stability options.
5. The Indexer receiving the Scan request will become the scan co-ordinator. It requests the local Index Manager for latest index topology information for the index to be scanned.
6. Index Manager responds with the index topology information for the requested index.
7. Indexer request latest Stability Timestamp from the Index Manager.
8. Index Manager responds with the latest Stability Timestamp.
9. Based on consistency/stability options for the scan, Indexer would either poll all KV nodes for the latest timestamp(Session Consistency) and mark that as the Scan Timestamp or choose Stability Timestamp for the Scan (Scan Stability) or a nil scan timestamp for scanning the tip.
10. Indexer will send all participating indexers(identified in Step 5) the Scan request with Scan Timestamp.
11. Local indexers will run the scan on the tip or a snapshot based on the decision made in Step 9.
12. Scan results are returned to Scan co-ordinator.
13. Results are consolidated/aggregated as required. 
14. Results are returned to index client. Local indexer may start streaming the results to index client before all scans are finished for efficiency.
