---
layout: default
---

# Virtuoso, a Hybrid RDBMS/Graph Column Store

___

This paper discusses the design choices met in applying column store techniques under the twin requirements of performing well on the unpredictable, semi-structured RDF data and more typical relational BI workloads. Virtuoso started out as a row-wise transaction oriented RDBMS and transitioned to being an RDF graph store and subsequently started using column-wise storage and vectored execution. The main incentive for using column-wise storage was its excellent space efficiency, which suited Virtuoso's large RDF applications.


## Column Store Implementation

* Virtuoso supports both row-wise and column-wise stored tables. The table is simply the index (B-tree) on its primary key with the dependent part following the key on the index leaf. The table is sorted on its primary key.

* For row-wise indexes, the table row itself is stored on the index leaf corresponding to the primary key entry.

* In case of column-wise indexes, the table is divided into segments (horizontal partitions, thousands of rows each) and the index is sparse and row-wise at the top, with the leaf storing references to compressed columns of that segment.

* Virtuoso has a small page size (8K) as compared to other column stores to facilitate the storage of both row-wise and column-wise data structures in the same buffer pool. Smaller pages also means low latency for random inserts.

* Compression schemes are column dependent and auto-tuned by the database.

* Virtuoso doesn't have the concept of a table wide row number (row id). Instead, rows have value based keys which can be used as partitioning keys for large cluster-based databases. Different indexes can be partitioned according to different columns and even stored on different nodes.


## Vectored Execution

* Benefits of columnar vectored execution (improved cache locality, tight loops, improved utilization of CPU memory throughput) were found to be applicable to row store performance as well.

* Pipelined operators like join, which change cardinality (no. of rows) of the output and add columns to the output, maintain input to output row mappings so that tuples can be reconstructed at the end. Reconstructing a large number of rows together optimizes memory bandwidth.

* Virtuoso uses vectoring for index lookups too. Since density of hits in an index determines performance, long vectors are preferred, because new lookups don't have to be made frequently as long as the value to be fetched is in the same segment (or same row-wise leaf page). This is especially useful for large read queries.

* Large vectors are also useful for overcoming latency in a cluster based deployment (lesser number of data transfers between nodes). The tradeoff here is that vectors don't fit in L1 CPU caches anymore. Nonetheless, Virtuoso batches have 10k to 1M values (10 to 1000 times larger than VectorWise batches).


## Updates and Transactions

* Virtuoso supports row level locking. Row level locks can be escalated to a lock on the entire leaf page in case of large read queries with similar semantics.

* Read queries don't block on uncommitted columns. Pre-images of uncommitted columns are shown.
