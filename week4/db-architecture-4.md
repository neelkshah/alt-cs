---
layout: default
---

# Architecture of a Database System

___

## Section 4 - Relational Query Processor

Can be viewed as a single-user single-threaded task, concurrency is mananged by lower layers of the system. (Exception - buffer pool pages)

### Query parsing and authorization
   - Canonicalize table names to a four part form (server.database.schema.table)
   - Get table information from the Catalog Manager, used to check attribute references and data types of attributes.
   - User authorization can be deferred till execution time, benefits of doing so: 
      1. can impose row level security
      2. users can share query plans
   - Constraint check constant expressions (can also be deferred to execution time)

### Query Rewrite
   - Sometimes a logical component included in Parser or Optimizer
   - View expansion: replace views with actual tables and predicates, handle column references in views
   - Constant expression evaluation
   - Logical predicate rewrite: check for satisfiability of expressions, add transitive predicates to help the optimizer choose better query plans.
   - Semantic optimization: redundant join elimination is very useful in case of view based wide table implementations.
   - Subquery flattening, heuristic rewrites: rewrite subqueries in a optimizer friendly form (query normalization)

### Query Optimizer

Query optimizer's job is to transform an internal query representation into an efficient query plan which is basically data flowing through a graph of query operators. Query plans are generally compiled into some interpretable data structure (lightweight object or bytecode for example) which enables cross-platform compatibility.

Optimizer's functionality can be extended to:
1. Select query plan space e.g. left deep plans only
2. Estimate selectivity
3. Decide on algorithms for searching the plan space
4. Generate parallelizable plans
5. Auto tuning based on query workload (e.g. choosing which views to materialize)

#### Prepare and execute model
   - Databases can store query plans for repeated execution, especially useful for queries with fixed structure over predictable data.
   - Plans for dynamically built queries can be stored too and used when similar queries are submitted.
   - There is an interesting tradeoff between predictable performance across invocations and optimal performance per invocation.

### Query Executor
   - Query executor on interpretable plans works on the iterator model (volcano-style). All relational operators are implemented as subclasses of the iterator class.

#### Iterators
   - Iterators couple dataflow with control flow i.e. a tuple is returned to a parent in the graph at the same time when control is returned. Which means a single DBMS thread is sufficient for executing the entire query graph.
   - Single threaded iterators can be extended to support parallel query execution.

#### Where is the data stored?
   - In flight tuples are stored in Buffer Pool pages (BP tuples) or copied into memory heap (M tuples)
   - Each iterator is allocated a fixed number of tuple descriptors (one for each input and one for output). A tuple descriptor is an array of column references where each reference is an actual memory address reference of a tuple and a column offset in that tuple.
   - Using M tuples exclusively is a performance problem because copying data in and out of memory heap is a bottleneck, whereas directly referencing to BP pages results in pinning a page for a long time in BP memory which might cause page replacement issues for high loads.

#### Handling DML queries

   - SQL semantics dictate that a single SQL statement should not be able to see its own updates, careful implementation is needed to ensure this.
   - ***The Halloween problem*** is an example of what can go wrong. The problem arises on statements like 
      >give everyone whose salary is under $20K a 10% raise.
   
   - If a plan uses an index scan iterator over salary field piplined into an update iterator, it results in all low-paid employees receiving repeated raises until they earn more than $20K.
   - One way to avoid this problem is not using indexes on the columns being updated, but this can be quite inefficient.
   - Another solution is to store IDs of all tuples qualifying for updates into a temporary file and scanning the file to fetch tuples to be fed to the update operator. This works well when only a few tuples are to be updated because, in that case, the temporary file is small enough to fit entirely in the buffer pool.


### Access methods
- Access methods are the routines that manage access to the various disk-based data structures that the system supports. These typically included unordered files (“heaps”), and various kinds of indexes e.g. hash indexes for equality lookups, multi dimensional R-tree indexes, Generalized Search Tree (GiST), RD-trees for text data.
- The API that access methods provide is the iterator API, with the `init()` method extended to accept a ***search argument (SARG)***.
- There are two reasons for passing SARGs into the access method layer:
   1. Accessing indexes like B+-trees require SARGs
   2. Unnecessary pinning/unpinning of pages and memory copy/delete can be avoided. If SARGs were checked by the routine that calls access methods, the method needs to either pin a page in the buffer pool or copy a tuple into memory in order to return a handle to the calling routine, an activity which might be a waste if the tuple doesn't satisfy the SARG. Therefore, it is better if the SARG is checked by the access method each page at a time so that only tuples that satisfy the SARG can be returned.
- Rows in base tables can be referenced by using their physical address on the disk or the row primary key. Using physical address is problematic in case an update requires the row to be moved to a different page. On the other hand, referencing by primary key is slower overall. Both physial address and primary key can be stored to be used in different cases.

### Data Warehouses
- Data warehouses are large historical databases for decision-support that are loaded with new data on a periodic basis.
- The nature of data stored in warehouses is very different as compared to data in an OLTP system and, as a result, requires different schema.
- Issues with data warehouse environments are:
   1. Support for bitmap indexes which are storage efficient but expensive to update.
   2. Fast loading of data in warehouses
   3. Materialized views - selecting the views to materialize (auto-tuning), maintaining fresh views, using these views for ad hoc queries
   4. Special support for predictable aggregate queries - data cubes
   5. Optimizing Snowflake schema queries