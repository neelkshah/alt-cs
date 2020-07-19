---
layout: default
---

# MonetDB: Two Decades of Research in Column-oriented Database Architectures

___

This paper presents a summary of the MonetDB system, main design points that shaped it and possible research areas in it. 


## MonetDB Design

MonetDB is designed primarily for data warehouse applications (OLAP) - intelligence queries and decision support. MonetDB mainly focuses on analytical and scientific workloads that are read-dominated and where updates mostly consist of appending new data to the database in large chucks at a time (batch updates). As we know from the column-store paper, MonetDB was one of the first databases to completely do away with row storage and design an execution engine specifically for column wise stored data.


### Physical Data Model

* Data is vertically fragmented and each column is stored in a seperate table known as a BAT (Binary Association Table). Each row of a BAT is a (surrogate, value) tuple.

* Surrogate is essentially a positional id for the row to which a column value belongs. Base tables don't need to materialize surrogates and use implicit positions.

* MonetDB uses typed C arrays to store columns with fixed-width data types. For variable width data like strings, dictionary encoding is used. All distinct values in the column are stored in a BLOB and individual tuples hold an integer array with references to BLOB positions. Surprisingly, this is the closest MonetDB comes to compressing column data.

* Late materialization is used i.e. tuples are reconstructed at the very last stage of query execution.


### Execution Model

* The core of MonetDB's execution engine is a low-level, two-column BAT algebra. Each operator is mapped to a simple assembly language instruction (MonetDB has a language of its own - MonetDB Assembly Language or MAL).

* Complex N-ary operations are broken down into a sequence of two-column BAT algebra operators, each working on entire columns at a time. Therefore, complex expression interpretation is avoided.

* The BAT algebra operators run tight for-loops without function calls that create high instruction locality which eliminates the instruction cache miss problem and leave room for compiler optimizations like loop pipelining.

* There is the problem of intermediate result materialization which can be solved, somewhat, by storing and reusing materialized results.


### System Architecture

MonetDB has a modular architecture consisting of:

* #### Frontend

    * This layer supports user-level data model and query language. Frontend is also responsible for mapping user-level data model to BATs and translating query language to MAL.

    * Before translation, queries are parsed into an internal representation upon which *strategic optimizations* are applied to reduce the size of intermediate results. These optimizations include heuristics like pushing down filter operations and using join indexes.

* #### Backend

    * This layer has the MAL optimizer framework and an interface to the kernel in the form on MAL interpreter.

    * MAL optimizers apply programming-language-like *tactical optimizations* on the MAL code passed down by the frontend.

* #### Kernel

    * The kernel provides highly optimized implementations of BAT relational algebra operators. Each operator can choose among multiple algorithm implementations based on input's properties at runtime (since full inputs are available at runtime). MonetDB creators call this *operational optimization*. For example, Select (filter) operator can use binary search in case of a sorted BAT, or an existing hash index or plain linear scan.

## MonetDB Research

Highlights of column oriented research in the context of MonetDB.

* #### Hardware-conscious Database Technology

    For automatically tuning cache-conscious algorithms on different types of hardware, the authors developed a cost model that takes the cost of memory access into account. The key idea is to abstract data structures as data regions and model the complex data access patterns of database algorithms in terms of simple compounds of a few basic data access patterns.

* #### Vectorized Execution and Compression

    VectorWise, developed by the same research group, addressed the problem of full materialization of intermediate results and incorporated column-wise data compression schemes.

* #### Reusing Intermediate Results with Recycler

    The Recycler project adaptively stores and reuses materialized intermediate results when possible e.g. when a select operator is covered by a stored intermediate of a past query. Intermediates are kept around as long as they fit in the allocated space for the Recycler and as long as they are hot.

* #### Adaptive Indexing and Database Cracking

    * MonetDB research pioneered Database cracking that allows on-the-fly physical data reorganization as a collateral effect of query processing. This solves the problem of dynamic data storage environments where traditional approaches to index building and maintainance cannot apply because: 1) there is little idle system time to carry out these tasks; and 2) query and data workloads keep changing constantly.

    * Variants of database cracking include - 
        1. Sideways cracking:
            Propagate cracking in one (leading) column to other columns in a lazy manner.
        2. Partial cracking:
            Put a storage bound on the auxiliary structures used by the cracking algorithm.
        3. Stochastic cracking:
            Randomly perform index refinement during query processing across various workloads.

* #### Data Cyclotron

    Applies mainly to distributed memory systems with DMA network facilities. The Data Cyclotron architecture creates a turbulent data storage ring through distributed main memory. Queries assigned to individual nodes interact with the storage ring by picking up data fragments that are flowing around.

* #### Adaptive Sampling and Data Exploration with SciBORQ

    In modern applications, not all data is equally useful all the time. A strong aspect in query processing is exploration. However, querying a multi-terabyte database requires a sizeable computing cluster, while ideally the initial investigation should run on the scientist’s laptop. SciBORQ is a framework for data exploration that works by sampling to provide biased snapshots of data.

* #### Stream Processing

    In the DataCell project, a stream processing engine is designed on top of MonetDB. It allows for window based processing, fast responses as data arrive and indexing on continuous streaming data.

* #### Graph Databases and Run Time Optimization

    To overcome the challenges faced by state of the art query optimizers, project ROX by incorporating the optimizer in the query execution phase


## Explore:

* [Data Cyclotron](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwiVmOzL5dbqAhXDH7cAHd5zA0kQFjABegQIBRAB&url=https%3A%2F%2Fopenproceedings.org%2F2010%2Fconf%2Fedbt%2FGoncalvesK10.pdf&usg=AOvVaw0jCoUalNNXMrMNgv4ewRFQ) sounds cool
* So does [SciBORQ](https://research.vu.nl/en/publications/sciborq-scientific-data-management-with-bounds-on-runtime-and-qua)