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

## A single global collection

A single collection, e.g. called: biblio.

* Docs: 200M
* Size: 400G
* Assumed number of ok shards: 200, 2G per shard

## One collection per index (main, ai)

## One collection per source

## One collection per technical collection

