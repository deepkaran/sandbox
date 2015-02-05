
##Secondary Index Deployment Options

####Overview

[Deployment Plan Design Doc](https://docs.google.com/document/d/1z0C7OodlagDnesvmL6OwbNcnpCrbkN8T_K4_Y6PKwgk/edit#heading=h.jyuvpp7j9swu) provides an overview of this feature.



####Examples of Deployment Plan Usage

#####Setup: 
2 node setup  
172.16.1.174:9000 - kv+index+n1ql 
127.0.0.1:9001    - index 

1. Create Index On A Node With Deployment Plan

CREATE INDEX index_abv 
ON `beer-sample`(abv) 
USING GSI 
WITH `{"nodes": ["172.16.1.174:9100"]}`

nodes parameter accepts the indexAdmin port on the index node which
can be discovered under nodeServices url:
http://172.16.1.174:9000/pools/default/nodeServices
"indexAdmin":9100


CREATE INDEX index_type
ON `beer-sample`(type) 
USING GSI 
WITH `{"nodes": ["127.0.0.1:9106"]}`

http://localhost:9001/pools/default/nodeServices
"indexAdmin":9106

2a. Create Index On A Node In Deferred Mode

CREATE INDEX index_abv 
ON `beer-sample`(abv) 
USING GSI 
WITH `{"defer_build": true}`

CREATE INDEX index_type
ON `beer-sample`(type) 
USING GSI 
WITH `{"defer_build": false}`

2b. Build A Deferred Index

BUILD INDEX ON `beer-sample`(index_abv, index_type) USING GSI;

Note - In this case index deployment is chosen randomly by Indexer.

3a. Create Index With Both Deployment and Deferred Mode Option

CREATE INDEX index_abv 
ON `beer-sample`(abv) 
USING GSI 
WITH `{"nodes": ["172.16.1.174:9100"], "defer_build": true}`

CREATE INDEX index_type
ON `beer-sample`(type) 
USING GSI 
WITH `{"nodes": ["127.0.0.1:9106"], "defer_build": true}`

3b. BUILD INDEX ON `beer-sample`(index_abv, index_type) USING GSI;

