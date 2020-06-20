Architecture of Database Systems
================================

Process Models
--------------

A DBMS Worker is the thread of execution in the DBMS that does work on behalf of a DBMS Client. A 1:1 mapping exists between a DBMS worker and a DBMS Client: the DBMS worker handles all SQL requests from a single
DBMS Client. The DBMS client sends SQL requests to the
DBMS server. The worker executes each request and returns
the result to the client.

1. Process per worker mapping

Gets OS support for scheduling and handling standard bugs like memory overruns. Sharing in-memory data structures across workers is a challenge. These must be explicitly allocated in OS-supported shared memory accessible across worker-processes. Scaling is a problem as processes have a large state to be stored (as compared to threads), and context-switching is expensive.

2. Thread per worker mapping

Each connection is allocated a new thread and the SQL request is executed entirely by that thread which runs a DBMS worker. Threads run within the multi-threaded DBMS process. The OS does not protect threads from each other's memory overruns and stray pointers. Thread APIs across OSs are not necessarily the same. All of this makes implementation and debugging difficult.

3. Process Pool

A definite number of processes are part of the pool. This reduces the memory overhead. After execution, processes are returned to the pool for reuse. The size of the pool can be dynamic.
