---
layout: default
---

# Section 1

***

Introduction, process models, models of parallelism

***

## Introduction

The relational data model developed in 1970 is the most commonly used data representation model. There are three layers that make up this model, in increasing order of abstraction are:
1. Physical schema which specifies exactly how data is stored in files and indexed in memory
2. Conceptual schema defines the logical relations between records/objects stored in the db
3. External schema are a collection of views and relations defined by users.

DML (Data Manipulation Language), DDL (Data Definition Language) are used to manipulate and represent data respectively. DBMS must have a locking protocol so that concurrent queries can run while being consistent. It should also recover from crashes usually by logging writes, this is known as Write-Ahead Log (WAL).

Why can't files be used as a DB?
1. Large files are not meant for computation because of memory limits.
2. Share access only depends on password which is not flexible enough to meet the needs of users.
3. Specials programs for each kind of query.

***

## Architecture of a Database System

### Process models

LWT threads are application layer threads managed by the program. They are very light weight and do not required kernel mode switch which makes them more efficient. The scheduling is handled by the application.

There are three main variants, with sub-variants in each,
1. Worker to Process mapping
    a. Simplest form, easily portable and to debug but not scalable
2. Worker to thread mapping
    a. Worker to OS thread
    b. Worker to LWT thread
3. Worker to Pool mapping
    a. Worker to process pool mapping
    b. Worker to thread pool mapping

All the sychronization is done using locks and shared data structures in the heap. More issues arise with multi-core systems but those are not described. There is also admission control mechanism to ensure that compute resources are not overwhelmed. The main tradeoffs between the three variants is between portability, scalability of default OS implementations and complexity of state management.

A more advanced trend is to split a query into a subqueries and process them in parallel using multiple execution engines deployed in a pool like manner.

***

### Models of parallelism

Shared Nothing - Disconnected nodes which access data across a high speed network. Data is horizontally partitioned across many nodes. Executing queries requires sychronisation and communication between nodes. This method also uses a technique called chained declustering, which copies tuples between multiple nodes to make data redundant (kind of like RAID architecture).
Shared Disk - Disconnected nodes which share the same disk, not *prevalent in very large databases because of need for partitioning the disk*.
Shared memory - consists of multi-core processors sharing memory. It has two variants
1. Uniform Memory Access
2. Non-uniform Memory Access

The concept of data centres is emerging with thousands of commodity servers and automated administration. This requires a lot of redundancy to handle node failure.
1. replication at storage level
2. data replication at the database storage engine level
3. redundant execution of queries by the query processor (interesting so basically a duplicate db running in parallel)
4. redundant db requests from client side

***

Column-oriented stores, introduction, trends and trade-offs

***

## Introduction

Data is vertically partitioned and each column is stored separately on disk. Benefits are in terms of utilization of memory and IO bandwidth. The main downside is interms of large seek times for finding individiual records, specially for wide tables. This cost is amoritized if the number of records being read is large. Some optimizations:

1. Virtual IDs (Disk): Fields values are stored in a fixed-width dense array. The offset can be easily calculated from the array index. However compression is commonly used to make storage efficient, making storage of non-fixed-length.
2. Block oriented (CPU cache): Passing records in chunk to make use of cache.
3. Late materialization (Memory IO): Compressed data is decompressed only when absolutely necessary greatly improves memory bandwidth
4. Column specific compression (Disk storage): where each column can be compressed differently
5. Efficient joins (??): as they are performed as late as possible
6. (Compute) Redundant representation of column in different sort orders
7. Database cracking (Compute): columns are sorted lazily and in chunks when queries are executed on them, reducing upfront cost
8. Efficient loading (IO) using a write buffer to store uncompressed data
