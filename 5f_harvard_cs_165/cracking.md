# Database Cracking

---

self-organizing databases, column organized datastores, sideways cracking

---

## Quick Refresher

[Database indexing](indexing.md)

## Introduction

Workload is not always stable. An index is created with a workload profile in mind. You spend resources to create and maintain indices. One problem in CS is that we build a data structure assuming we know our exact need and its exact use, and expend time and resources building said structure. We do on-the-fly physical reorganization of data in-memory with every query.

## Column Stores

Fetch required columns only. Columns are of a fixed length - they can be compressed very efficiently. Moreover, since they are of fixed length, they essentially form arrays. Now sequential access becomes much better with improved temporal locality.

Columns are brought in one-at-a-time. For example, if a query selects col A and an aggregation of col B, MonetDB first fetches A, does a select, fetches B, does select and aggregate, and then returns the two columns put together. The upside is that there is one function call per column (as compared to one per tuple for row stores). The downside is that intermediate results have to be materialized. (Vector wise implementation tries to find an intermediate solution).

Row stores operate on the *volcano model* - each processor asks for a tuple, applies all operators, asks for the next tuple and so on. Note that memory needs are low here. But you do carry the entire tuple for each operator. Column stores operate on the *block model*. What reaches the CPU is only the data you need.

Implementation of operators is a sensitive task. How a select is implemented (scanning, hash table, etc) depends on a number of factors - data distribution, resource constraints, and most importantly in the case of row stores, on the output of the previous operator. In column stores, this optimization at the operator level can be deferred as far back as possible, since at each step, we know the contents in the column and everything about them.

Tuples need to be stitched together from different columns. Tuples can be glued together immediately after fetching from disk (early reconstruction). This retains the I/O benefit of fetching data faster that column stores offer. Post fetching, we basically have a row store. Tuple construction can also be done lazily. This allows for vector-like processing, CPU and cache optimization and bulk processing (as opposed to volcano). But now, we have to rewrite the DB kernel accordingly. Late reconstruction can be orders of magnitude better than row stores even if all columns are queried.

We have seen the selection example (select a from table where 9 <= a <= 16). The data is physically reorganized as per the limit specified in the query. Now these changes need to be propagated to the other columns as well. This is done in a lazy manner. The key idea is that I/O must always be performed in a sequential manner. If there is a query on A-B, B is re-organized. If there's a query on A-C, C is re-organized, and so on. There's always a leading column that has the latest re-organization. This is referred to as sideways cracking: cracking that is propagated sideways.
