# Column-Stores

* Column-store systems completely vertically partition a DB into a collection of individual columns that are stored separately.
* If a single record needs to be fetched, column stores need multiple seeks, whereas a row store can do it in a single seek. If there are many records to be accessed, the seek time gets amortized over records. As more and more records are accessed, transfer time begins to dominate seek time and column stores begin to perform better than row stores. Hence columns stores are typically used in analytic applications, with queries that scan a large fraction of individual tables and compute aggregates or statistics over them.

## Specific Ideas

1. Virtual IDs: Explicitly representing a key with each value stored in the column bloats the size of the data on disk, and makes I/O less efficient. Instead, we could use the offset of the tuple in the column as a virtual identifier. Attributes can be stored as fixed-width dense arrays, and each record can be stored at the same array position across columns. This makes it very simple to access a record based on its offset. The *i*-th value in column *A* resides at the location *startOf(A) + i\*width(A)*. Another major advantage of column stores is improved compression ratio. Now it is entirely possible that a good compression algorithm compresses data in a non-fixed-length way, such that data cannot be stored in an array. This makes it a trade-off between simplified sequential access and greater compression.
2. Block-oriented and vectorized processing: In row-stores, multiple function calls are made to process a tuple at a time. In column stores, since we process a single column at a time, we make a single call to the column. Values are chunked into vectors. These vectors are processed one at a time, making use of bitwise operation and SIMD hardware support.

![Column Store Features](https://github.com/neelkshah/alt-cs/blob/master/5f_harvard_cs_165/resources/col_features.png "Column-store Features")

