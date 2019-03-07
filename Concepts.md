# Key Concepts

SolrCloud nomenclature.

## Physical and Logical

Logical concepts are concerned with documents and their grouping, physical concept relate to their implementation.

### Logical

* one **cluster** hosts multiple **collections**, 1:n
* one **collection** contains **documents**, 1:n
* one **collection** can be **partitioned** into **shards**, with each shard
  containing a **subset** of the documents

### Physical

* one **cluster** consists of one or more **nodes**
* one **node** can host multiple **cores**
* each **core** is a physical **replica** of a logical **shard**

Every replica uses the configuration for the collection it is part of. The
number of replicas determine redundancy, performance and concurrent searches.

