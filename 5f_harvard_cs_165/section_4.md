# Section 4

***

Architecture, relational query processing

***

## Relational Query Processor

Can be viewed as a single-user single-threaded task, concurrency is mananged by lower layers of the system. (Exception - buffer pool pages)

### Query parsing and authorization
   - Canonicalize table names to a four part form (server.database.schema.table)
   - Get table information from the Catalog Manager, used to check attribute references and data types of attributes.
   - User authorization can be deferred till execution time, benefits of doing so :- can impose row level security, users can share query plans
   - Constraint check constant expressions (can also be deferred to execution time)
### Query Rewrite
   - Sometimes a logical component included in Parser or Optimizer
   - View expansion: replace views with actual tables and predicates, handle column references in views
   - Constant expression evaluation
   - Logical predicate rewrite: check for satisfiability of expressions, add transitive predicates to help the optimizer choose better query plans.
   - Semantic optimization: redundant join elimination is very useful in case of view based wide table implementations.
   - Subquery flattening, heuristic rewrites: rewrite subqueries in a optimizer friendly form (query normalization)
