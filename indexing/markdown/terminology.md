##Terminology Document

This document defines the terminology used in Secondary Indexes.

- High-Watermark Timestamp
- Stability Timestamp
- Restart Timestamp
- Mutation Queue
- Catchup Queue
- Stability Snapshot
- Persistent Snapshot
- Partition
- Slice


**Forward Index** - Main persistence structure for secondary index. This typically
 has the format of <secondary_key, docid> and allows lookup/scan queries based on the
 secondary key.
 
 **Back Index** - Auxiliary persistence structure for secondary index. This typically
 has the format of <doc_id, secondary_key>. The main usage of a back-index is in 
 maintainence of the Forward Index when the source document gets mutated. It facilitates
 in looking up the old secondary key associated with a given document. This old secondary
 key can then be looked up and replaced/deleted based on the mutation.
 
 **Tearing Read** - Tearing Read is a problem of non-isolated results in distributed indexes
 under concurrent modification. For a detailed explanation of the problem, please read 
 [Reference 3](https://docs.google.com/document/d/1Y_aXMUBzEvLf8PO8CJYv5eYiQmKsNYzMr6Fq30Cl6xg/edit#heading=h.phqy8trsrvu4).
