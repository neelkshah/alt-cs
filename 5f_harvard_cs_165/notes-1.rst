################################
Architecture of Database Systems
################################

**************
Process Models
**************

A DBMS Worker is the thread of execution in the DBMS that does work on behalf of a DBMS Client. A 1:1 mapping exists between a DBMS worker and a DBMS Client: the DBMS worker handles all SQL requests from a single DBMS Client. The DBMS client sends SQL requests to the DBMS server. The worker executes each request and returns the result to the client.

Process per worker mapping
   Gets OS support for scheduling and handling standard bugs like memory overruns. Sharing in-memory data structures across workers is a challenge. These must be explicitly allocated in OS-supported shared memory accessible across worker-processes. Scaling is a problem as processes have a large state to be stored (as compared to threads), and context-switching is expensive.

Thread per worker mapping
   Each connection is allocated a new thread and the SQL request is executed entirely by that thread which runs a DBMS worker. Threads run within the multi-threaded DBMS process. The OS does not protect threads from each other's memory overruns and stray pointers. Thread APIs across OSs are not necessarily the same. All of this makes implementation and debugging difficult.

Process Pool
   A definite number of processes are part of the pool. This reduces the memory overhead. After execution, processes are returned to the pool for reuse. The size of the pool can be dynamic.

Shared Data
   In the *thread-per-worker* model, data sharing is easy since threads share the same address space. In the other models, shared memory is used. In all three models, data needs to be moved from DBMS to client. Requests need to be moved into the server processes and results need to be moved back out. Various buffers are used, the two major types being **disk I/O buffers** and **client communication buffers**.

Disk I/O buffers
   All persistent database data is staged through the **DBMS buffer pool**. With thread per DBMS worker, the buffer pool is simply a heap-resident data structure available to all threads in the shared DBMS address space. In the other two models, the buffer pool is allocated in shared memory available to all processes. The end result in all three DBMS models is that the buffer pool is a large shared data structure available to all database threads/processes.
   As **log entries** are generated during transaction processing, they are staged to an in-memory queue that is periodically flushed to the log disk(s) in FIFO order. This queue is usually called the log tail. With thread per DBMS worker, the log tail is simply a heap-resident data structure. Either a separate process manages the log (efficient IPC mechanisms) or the log tail is allocated in shared memory like the buffer pool above.

Client Communication buffers
   SQL is typically used in a *pull* model. Client socket is used to enqueue results. More complex approaches use client side cursor caching and use the client to store results that are likely to be fetched in the near future instead of relying on OS buffers.

Lock Table
   The lock table is also shared by all workers and is shared in a manner similar to the buffer pool.

*********************
Parallel Architecture
*********************

Shared Memory
   Each processor is equidistant from and shares memory and caches.
