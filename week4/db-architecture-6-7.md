---
layout: default
---

# Architecture of a Database System

___

## Section 6 - Transactions

In a DBMS, the transaction storage manager has four deeply intertwined components,
1. A lock manager for concurrency control
2. A log manager for recovery
3. A buffer pool for staging database I/Os
4. Access methods for organizing data on disk

___

Concurrency is mainly concerned with ACID properties, specially maintaining the isolation between two queries working on the same piece of data. There are three main concurrency control models.

1. Strict two-phase locking: Transactions acquire a shared lock on every data record before reading it, and an exclusive lock on every data item before writing it.
2. Multi-Version Concurrency Control (MVCC): Transactions do not hold locks but are guaranteed a consistent view of the database at a certain point in time.
3. Optimistic Concurrency Control: No restrictions, transactions maintain history of changes and check for conflicts before committing. In case of conflict changes are rolled back.

Locks are logical mechanism to restrict access to any kind of resource physical or logical. A lock simply provides a place to check for names (that are already being used by something else). Hierarchical locking allows a single lock for an entire table and then row granularity locks for both efficiently and correctness.

Lock manager maintains two data structures namely,
1. A global lock table to hold lack names and their lock mode, and wait queue of of waiting requests (transactionID, mode)
2. A transaction table keyed by transactionID, with pointer to a transaction T's DBMS thread state and pointers to all of T's lock requests

This double mapping between locks and transactions allow efficient lock allocation and resolving conflicts by deleting locks for a transaction for aborted transaction.

There is more low-level in memory construct called a latch that is used to implement locks. Latches are also for access control for things like buffer pages. They are much faster but can cause deadlocks if not used properly

The problem of phantom tuples arises in many cases. Suppose a query says `GET AGE < 30 AND AGE > 10 in PERSON`. Most locking mechanism will ensure consistency and values satisfying these predicates will not be modified or delete by other queries. However they cannot prevent insertion. So a different query can insert new tuples where AGE value is 25, 26, etc. This means if the predicate is used multiple times in the same query it can give different results (more values) each time, because new records were added.

___

## Section 7 - Shared Components

___

Catalog Manager - Holds metadata about the system, for e.g. users, schemas, tables, columns, indexes etc. It is stored as a table and uses all the components that are used to manage actual data.

Memory Allocator - Memory is alloted for various uses apart from answering queries. Managing this memory is big burden. This handled by providing a level of abstraction over memory, which is called memory context. A single memory context contains regions of contiguous memory called memory pools. It also provides APIs that allows developers to use memory efficiently. There are numerous advantages to this,
1. Manages large pools of memory and reduces the number of calls to `malloc()`, `free()` thereby reducing OS overhead
2. Gives developers better control of temporal and spatial memory locality

Disk management systems that actually store data. This causes problems to DB designers often because they have non-uniform write times to different parts of disk. More importantly the RAID model, that is very common reduces performance, because it is designed for bytestream-oriented (UNIX files) storage. Chained declustering method is more suitable for DBs.

Replication services - Most feasible choice is log based replication
Administration, monitoring, and utilities

___

