# scintro

An introduction to SolrCloud setup and operations.

Official documentation:

* [SolrCloud](https://lucene.apache.org/solr/guide/6_6/solrcloud.html)

> SolrCloud is flexible distributed search and indexing, without a master node
> to allocate nodes, shards and replicas. Instead, Solr uses ZooKeeper to
> manage these locations, depending on configuration files and schemas. Queries
> and updates can be sent to any server. Solr will use the information in the
> ZooKeeper database to figure out which servers need to handle the request.

## Interactive setup

Covered in the documentation, [SolrCloud Example](https://lucene.apache.org/solr/guide/6_6/getting-started-with-solrcloud.html#getting-started-with-solrcloud).

* collection name
* number of shards
* number of replicas

As opposed to ElasticSearch, which is mostly API driven, Solr relies on
configuration files and directories (e.g. when [adding a new
node](https://lucene.apache.org/solr/guide/6_6/getting-started-with-solrcloud.html#GettingStartedwithSolrCloud-Addinganodetoacluster)).

> Lastly, the script will prompt you for the name of a configuration directory for your collection.

## Key concepts

### Logical

* a cluster hosts multiple collections
* a collection hosts multiple documents
* one collection can be partitioned into shards (each shard contains a subset
  of documents) - the number of shards determined a theoretical limit for the
number of documents and also has influence on parallelization of requests

### Physical

* one cluster is made of one or more nodes
* each node can host multiple cores
* each core in a cluster is a physical replica for a logical shard
* every replica uses the same configuration specified for the collection (it is part of)
* the number of replica that each has determines: level of redundancy (node
  failure), theoretical limit in the number of concurrent searches

### Questions

> Given 100M documents that would exceed the storage capacity of a single node,
is it possible to use SolrCloud with a single shard?

No. When your collection is too large for one node, you can break it up and
store it in sections by [creating multiple
shards](https://lucene.apache.org/solr/guide/6_6/shards-and-indexing-data-in-solrcloud.html).

## Sharding

Sharding is partitioning. There can be various partitioning strategies (e.g.
group things from the same country, or just use some hash on the unique key of
the document to mix documents).

Sharding was there before SolrCloud, but it was a bit different:

> Before SolrCloud, Solr supported *Distributed Search*, which allowed one query
> to be executed across multiple shards, so the query was executed against the
> entire Solr index and no documents would be missed from the search results.

> So splitting an index across shards is not exclusively a SolrCloud concept.
> There were, however, several problems with the distributed approach that
> necessitated improvement with SolrClouSharding was there before SolrCloud,
> but it was a bit different:

> 1. Splitting an index into shards was somewhat manual.
> 2. There was no support for distributed indexing, which meant that you needed
>    to explicitly send documents to a specific shard; Solr couldn’t figure out
>    on its own what shards to send documents to.
> 3. There was no load balancing or failover, so if you got a high number of
>    queries, you needed to figure out where to send them and if one shard died
>    it was just gone.

SolrCloud can distribute indexing and query ops, ZK manages failover, load
balancing, replicas for performance and robustness.

There are no masters or slaves in SolrCloud:

> Instead, every shard consists of at least one physical replica, exactly one
> of which is a leader.

ZK does leader election on a first-come-first-serve basis.

Leader is responsible for the indexing. After indexing the update is distributed to the replicas.

Question: Is it possible to get an 200 OK on a commit only after all replicas have been written?

### Document routing

When a collection is created, the `router.name` parameter allows to set the routing strategy.

### Questions

> When to set the routing strategy?

On collection creation.

> Which parameter controls the routing strategy?

The `router.name` parameter.

> What is the default routing strategy name?

It is called `compositeId`, it allows to route with a prefix. The prefix is
added at index time to the id field, e.g. `IBM!1234`, where `IBM` is a prefix.

> How many levels does the compositeId router support?

Two levels, e.g. `USA!IBM!1234`.

> How do you specify route at query time?

Use the `_route_` parameter, it can improve query performance.

### Large sets

Large sets of documents can be routed and use multiple shards, e.g.
`shard_key/num!document_id` will take num bits from the shard key to use in a
composite hash.

So "IBM/3!12345" will take 3 bits from the shard key and 29 bits from the
unique doc id, spreading the tenant over 1/8th of the shards in the collection.

### A field in the document specifying the shard

A field can be designated in the document to indicate the shard, via
`router.field` parameter. If the field is missing in the document, it will be
rejected.

### Shard splitting

How to decide on the number of shards at collection creation time?

> The ability to split shards is in the Collections API. It currently allows
> splitting a shard into two pieces.

### Clouds and Commits

In Solr, we usually commit after indexing (otherwise documents will not be
visible).

In SolrCloud it is [different](https://lucene.apache.org/solr/guide/6_6/shards-and-indexing-data-in-solrcloud.html#ShardsandIndexingDatainSolrCloud-IgnoringCommitsfromClientApplicationsinSolrCloud).

> Rather, you should configure auto commits with openSearcher=false and auto
> soft-commits to make recent updates visible in search requests. This ensures
> that auto commits occur on a regular schedule in the cluster.

Commit requests can ignored by Solr (as it is not feasable to update all client
applications).

> IgnoreCommitOptimizeUpdateProcessorFactory

```xml
<updateRequestProcessorChain name="ignore-commit-from-client" default="true">
  <processor class="solr.IgnoreCommitOptimizeUpdateProcessorFactory">
    <int name="statusCode">200</int>
  </processor>
  <processor class="solr.LogUpdateProcessorFactory" />
  <processor class="solr.DistributedUpdateProcessorFactory" />
  <processor class="solr.RunUpdateProcessorFactory" />
</updateRequestProcessorChain>
```

Include defaults, because chain replaces the default. It is possible to return
other status codes as well.

## Distributed Requests

Docs: [https://lucene.apache.org/solr/guide/6_6/distributed-requests.html](https://lucene.apache.org/solr/guide/6_6/distributed-requests.html)

> When a Solr node receives a search request, the request is routed behind the
> scenes to a replica of a shard that is part of the collection being searche

It is possible to limit the query to a number of shards (You have the option of
searching over all of your data or just parts of it.).

How to limit the shards, that are queried?

* [Docs](https://lucene.apache.org/solr/guide/6_6/distributed-requests.html#DistributedRequests-LimitingWhichShardsareQueried)

Example:
[http://localhost:8983/solr/gettingstarted/select?q=*:*&shards=shard1,shardo2](http://localhost:8983/solr/gettingstarted/select?q=*:*&shards=shard1,shard2),
a random replica is chosen - but it is also to specify down to replica level,
e.g.
[...shards=localhost:7574/solr/gettingstarted|localhost:7500/solr/gettingstarted](http://localhost:8983/solr/gettingstarted/select?q=*:*&shards=localhost:7574/solr/gettingstarted|localhost:7500/solr/gettingstarted),
mix and match (random replica, explicit list of replicas) is possible.

Configure various shard handlers:

```xml
<requestHandler name="standard" class="solr.SearchHandler" default="true">
  <!-- other params go here -->
  <shardHandler class="HttpShardHandlerFactory">
    <int name="socketTimeOut">1000</int>
    <int name="connTimeOut">5000</int>
  </shardHandler>
</requestHandler>
```

Default favors throughput over latency.

There are at least four options for document stats implementation, [docs](https://lucene.apache.org/solr/guide/6_6/distributed-requests.html#DistributedRequests-ConfiguringstatsCache_DistributedIDF_).

```xml
<statsCache class="org.apache.solr.search.stats.ExactStatsCache"/>
```

> How can a distributed dead lock occur?

The number of threads serving HTTP requests is smaller than the number of
shards. Each shard gets a request and distrbutes it to all other nodes, you
need at least as many threads as shards.

> Care should be taken to ensure that the max number of threads serving HTTP
> requests is greater than the possible number of requests from both top-level
> clients and other shards.

In order to limit the number of queries, SolrCloud can be instructed to prefer
local shards, if they are available, include `preferLocalShards=true` in the
query.



