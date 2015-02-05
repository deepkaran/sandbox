
##Secondary Index Deployment Options

####Overview

[Deployment Plan Design Doc](https://docs.google.com/document/d/1z0C7OodlagDnesvmL6OwbNcnpCrbkN8T_K4_Y6PKwgk/edit#heading=h.jyuvpp7j9swu) provides an overview of this feature.

####Syntax

```CREATE INDEX index_type
ON `beer-sample`(type) 
USING GSI 
WITH `{"nodes": ["node_addr"], "defer_build": true}`'```

**node_addr** refers to the couchbase cluster address for the index node ie. hostname:port(e.g. 127.0.0.1:8091, 127.0.0.1:9000 etc)

**defer_build** determines if index build is immediate or after an explicit build index command

**Note** Currently due to a bug the **node_addr** needs to have the **indexAdmin** port rather than the ns_server port. This port can be discovered for an Indexer Node from the nodeServices url (e.g.  http://127.0.0.1:9000/pools/default/nodeServices and look up indexAdmin port for the current node). If the indexAdmin port is 9100, the **node_addr** would be 127.0.0.1:9100.

####Examples of Deployment Plan Usage

#####Setup: 
 
**Node1**(172.16.1.174:9000) - kv+index+n1ql 

**Node2**(127.0.0.1:9001)    - index 

**Create Index On A Node With Deployment Plan**

```CREATE INDEX index_abv 
ON `beer-sample`(abv) 
USING GSI 
WITH `{"nodes": ["172.16.1.174:9100"]}`;```

nodes parameter accepts the indexAdmin port on the index node which
can be discovered under nodeServices url:
http://172.16.1.174:9000/pools/default/nodeServices
"indexAdmin":9100


```CREATE INDEX index_type
ON `beer-sample`(type) 
USING GSI 
WITH `{"nodes": ["127.0.0.1:9106"]}`;```

http://localhost:9001/pools/default/nodeServices
"indexAdmin":9106

**Create Index On A Node In Deferred Mode**

```CREATE INDEX index_abv 
ON `beer-sample`(abv) 
USING GSI 
WITH `{"defer_build": true}`;```

```CREATE INDEX index_type
ON `beer-sample`(type) 
USING GSI 
WITH `{"defer_build": true}`;```

```BUILD INDEX ON `beer-sample`(index_abv, index_type) USING GSI;```

Note - In this case index deployment is chosen randomly by Indexer.

**Create Index With Both Deployment and Deferred Mode Option**

```CREATE INDEX index_abv 
ON `beer-sample`(abv) 
USING GSI 
WITH `{"nodes": ["172.16.1.174:9100"], "defer_build": true}`;```

```CREATE INDEX index_type
ON `beer-sample`(type) 
USING GSI 
WITH `{"nodes": ["127.0.0.1:9106"], "defer_build": true}`'```

```BUILD INDEX ON `beer-sample`(index_abv, index_type) USING GSI;```

