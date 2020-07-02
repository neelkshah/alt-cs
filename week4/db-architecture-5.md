---
layout: default
---

# Architecture of a Database - Storage Management

***

storage, disk access, spatial and temporal effects, buffer management, OS

***

* The DBMS could either interact with low level block mode device drivers for disk access - **raw mode access**, or it could use the file system facilities provided by the operating system.
* This decision has spatial and temporal effects on the control the DBMS has over storage

## 5.1 Spatial Control

* Disk density doubles every 18 months, bandwidth grows as square root of density. But disk arm movement is terribly slow, with speeds increasing by approx. 7% a year. Sequential access is therefore of the essence.
* Since the DBMS knows it's workload better than the OS, it makes sense for the DBMS to have control over spatial positioning of data blocks in disk.
* The best option for the DBMS is to store data directly to raw device addresses. These typically correspond to physical proximity of storage locations.
* Since entire disk partitions are devoted to the DBMS they become unavailable to utilities (such as backup) that require a file system interface.
* Secondly, raw device addresses are often OS specific, which would make a DBMS difficult to port. However this problem has been solved in systems like RAID and SANs. We use virtual disk devices - the "raw" disk interface is intercepted by software applications that aggressively reposition data across disks.
* An alternative is to use a very large file in the file system. We assume that the file will mostly consist of physically contiguous locations by virtue of its size. The file is treated as a linear array of disk-resident pages.

## 5.2 Temporal Control (Buffering)

* A DBMS contains critical logic about when data is to be written to disk. Going through the OS means the scheduling algorithms used by the OS could confound this logic (silently).
* Problem 1: correctness (ACID). If the DBMS does not control writes, it cannot guarantee consistency. It cannot guarantee atomic recoveries. This is solved by Write-Ahead-Logging (WAL). The transaction is committed to the logging device before being acknowledged to the user. Actual data may be written later.
* Problem 2: performance. OSs are tuned for speculative reads (read colocated pages ahead of time) and delayed or batched writes (read ahead, write behind). The sequential order of data is known to the DBMS and might not correspond to physical colocation. For example, discontiguous nodes of a B+-tree.
* Problem 3: double buffering. Data moves from disk to OS buffers to DBMS buffers and then to process (and in the reverse order for opposite direction of data flow). OS buffers are redundant in this process. Copying data in-memory is therefore an unnecesarily costly bottleneck. I/O is generally not a major bottleneck (it is resolved through horizontal scaling - more RAM, more disks). Therefore, unnecessary CPU work like in-memory copying becomes a major bottleneck (remember RAM access speed trails Moore's law significantly). (`mmap` calls are used to bypass the OS buffers)

## 5.3 Buffer Management

* Buffers consist of a dynamic number of fixed-size frames. Each frame corresponds to a DBMS data block. (This discussion is the same as pagination in OS)
* A hash table keeps track of frame numbers, their locations in memory and on backing disk, and associated metadata.
* The metadata includes a dirty bit and pin count.
* Page replacement policies are used to evict pages when the buffer pool is full.
* Typically, LRU variants are used.
