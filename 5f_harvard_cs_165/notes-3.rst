#################
Database Indexing
#################

`UC Davis PDF<https://web.cs.ucdavis.edu/~green/courses/ecs165a-w11/7-indexes.pdf>`_

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

.. note::

   The search key and primary key need not be the same. For example, primary key could be order_id, but search key could be a timestamp field.

