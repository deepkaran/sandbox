##Deployment Options

Below diagram depicts the production deployment for Secondary Indexes. <br>
KV, Index and Query should be running in their own clusters. 
<br>

![](https://rawgithub.com/deepkaran/sandbox/master/indexing/images/Deployment.svg)

<br>
In development environment, KV, Indexer and Query can be colocated on the same node.
