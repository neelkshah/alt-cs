---
layout: default
---

# Questions, and hopefully, Answers

Q. Could compression in column-stores affect algorithm complexity of DB operations?

A. No. Complications would arise mostly from combinations of records. However, we keep records separate. Moreover, we are interested in compression-aware scanning. Compression is intended to affect storage. Compression algorithms include run-length encoding, bit-vector encoding, dictionary encoding, frame of reference encoding, differential encoding, and so on.

***

Q. What is the patching technique in column-stores?

A. Compression techniques often rely on a certain uniformity in the data. For example a hash table or frequency partitioning will perform much better if there are a lot of similar values. To optimize the fast path (i.e. common values) patching is used. In this technique, using some heuristic certain values are classified as outliers. These are stored separately in a linked list. Decompression first compresses the columns and then traverses the linked list and patches these exceptional values into the decompressed output.

***

Q. What does CURSOR STABILITY isolation mode do?

***

Q. How does Next-Key locking solve phantom tuple problem?

A. The phantom tuple problem is caused by inserts or updates that are applied after the predicate of a previously fired query has been evaluated, and before the this previous query has been fully executed. The new operation could introduce phantom tuples that affect the correctness of the already-running query. Locking is typically row-based, and therefore does not address the problem caused by an altogether new tuple being inserted, since there is no way we could have taken a lock on a row that was yet to be inserted. Next-key locking solves this problem to some extent. In this mechanism, we take a lock on a group of index records instead of rows. For example, consider and index containing the following search key ranges: (-infinity, 10], [11, 13], [14, 20], [21, infinity). If a query needs to evaluate the predicate `AGE > 18` (supposing these are ages), we take a lock on the **next key** that appears in the index, and its previous interval. The next key (relative to 18) is 20, and the previous interval is [14, 20). In other words, we take a lock on the range of index values containing the limiting value of the predicate. This interval could be an open-ended supernum too. Now, if we were to receive a new query `insert age 19`, it would have to wait for the lock on the range to be released. Other queries might succeed (`insert age 50`), but we have to live with this limitation, since the only alternative is to take a lock on the entire table. 
