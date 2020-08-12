---
layout: default
---

# 4. Vectorwise: Beyond Column Stores

## Key ideas

* Maintains a single copy of all data
* I/O - NSM/PAX vs DSM/PAX
* Disk storage - compressed data with high decompression rates
* DDL allows users to declare one index per table in order to determine the physical tuple order
* Maintains (automatically) MinMax indices on all columns to store simple metadata for a given range of values. This is particularly advantageous if there are correlations between attribute values and tuple position
* To assist with concurrent scan intensive queries, table scans accept data out-of-order, and a buffer manager determines tuple order at runtime.
* Update mechanism is differential in nature and uses a three layer PDT design - a small PDT private to the transaction, a shared CPU-cache resident PDT, and a potentially large RAM-resident PDT to provide snapshot isolation.
* Query processing exploits SIMD execution alongwith environment specific optimization
* Future - query execution on compressed data, JIT compilation

