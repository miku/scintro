# Scenarios

These are a few scenarios given the following situation:

* hundreds of data sources (SID), with varying number of documents, from few to
  hundred million
* thousands of collections (TCID), each belonging to exactly one source, size:
  from few to few million docs
* individual update frequencies from daily to monthly to infrequently (UP)
* need to change one or more fields in a documents on a running infrastructure (MOD)
* indexing schema needs updates (SCHEMA), and a flip-switch method would be interesting (FLIP)

In short: SID, TCID, UP, MOD, SCHEMA, FLIP.

## Scenarios

The following scenarios are examples, the relate to number of docs, physical
index size, estimated number of ok shards (ENOOS)

We assume that machines are connected via 1Gbps (125 MB/s), which means that 1G
can be transferred in 8s, 5G in 40s and so on.

Cluster stability will also depend on the data that needs to be moved around.

* TODO(miku): Create spreadsheet with various scenarios

## A single global collection

A single collection, e.g. called: biblio.

* Docs: 200M
* Size: 400G
* Replicas: 1
* ENOOS: 200 (2G per shard)
* Total size: 800G

The shards do not correspond at all to SID and TCID. A copy, e.g. for testing
must be done per collection, which is large.

## One collection per index (main, ai)

### Main

* Docs: 10M
* Size: 100G
* ENOOS: 20 (5G per shard)

### AI

## One collection per source

## One collection per technical collection

