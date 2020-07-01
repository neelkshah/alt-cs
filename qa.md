---
layout: default
---

# Questions, and hopefully, Answers

* Could compression in column-stores affect algorithm complexity of DB operations?

A. No. Complications would arise mostly from combinations of records. However, we keep records separate. Moreover, we are interested in compression-aware scanning. Compression is intended to affect storage. Compression algorithms include run-length encoding, bit-vector encoding, dictionary encoding, frame of reference encoding, differential encoding, and so on.

* What is the patching technique in column-stores?

A. Compression techniques often rely on a certain uniformity in the data. For example a hash table or frequency partitioning will perform much better if there are a lot of similar values. To optimize the fast path (i.e. common values) patching is used. In this technique, using some heuristic certain values are classified as outliers. These are stored separately in a linked list. Decompression first compresses the columns and then traverses the linked list and patches these exceptional values into the decompressed output.

* What does CURSOR STABILITY isolation mode do?

* How does Next-Key locking solve phantom tuple problem?

