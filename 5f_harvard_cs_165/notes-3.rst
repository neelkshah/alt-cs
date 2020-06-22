#################
Database Indexing
#################

`UC Davis PDF <https://web.cs.ucdavis.edu/~green/courses/ecs165a-w11/7-indexes.pdf>`_

************
Introduction
************

Search key
   Attribute or combination of attributes used to look up records in a file

Index file
   Consists of records (index entries) of the form (search key value, pointer to block in data file)

Index files are typically much smaller than original data.

There are mainly two types of indices:
   1. Ordered indices - search keys are stored in sorted order
   2. Hash indices - search keys are uniformly distributed across hash buckets

Indexing techniques are evaluated on the basis of:
   1. Access types that are supported (select specific value, select range)
   2. Access time (index entry --> record)
   3. Insertion time (record --> index entry)
   4. Deletion time (record --> index entry)
   5. Space and time overhead of (building and) maintaining index

****************************
Single Level Ordered Indices
****************************

In an ordered index file, index entries are sorted by search key values.

Primary index
   In a sequentially ordered data file, the index whose search key specifies the sequential order of the file. For a relation, there can be at most one primary index.

``The search key and primary key need not be the same. For example, primary key could be order_id, but search key could be a timestamp field.``

Secondary index
   An index whose search key is different from the sequential ordering of the data file.

Index files could be dense or sparse. A dense index file would consist of entries for each search key value in the data file. On the other hand, a sparse index file would contain only a subset of the search key values. This reduces the space overhead and makes insertion and deletion easier. But it is slower at diresctly locating records as compared to a dense index.

A secondary index also consists of entries of type (search key value, pointer to data block) except the data file is not sorted by the search key. Therefore I/O is not sequential and takes time. Moreover, secondary indices MUST be dense since data is not sorted by search key and we need to maintain the pointer location of each individual block. Thus there are multiple index entries for the same search key. In order to avoid this, we can set up an additional level of indirection by making each search key value point to a single bucket that contains all the pointers associated with the search key. This helps avoid the problem of having multiple index entries with the same search key value.

Primary indices vs. Secondary indices
1. Secondary indices have to be dense
2. Sequential scan using primary index is efficient, but using secondary index is expensive.

*******************
Multi-level Indices
*******************

If a primary index does not fit in memory, using it makes record access expensive. To reduce the number of disk accesses to index entries, we treat the primary index (which is on disk) as a sequential file, and construct a sparse index over it. Therefore we now have a sparse outer index over a (possibly dense) inner primary index. This tiering can be done until we get to a level where the outermost index suitably fits in memory. Indices at all levels, must however, be updated upon insertion or deletion of records from the underlying data file.
