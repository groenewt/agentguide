# Concept: DuckDB as Unified Substrate
## Single in-process DuckDB instance. Three writer threads, N reader threads. .cursor() per thread for thread safety. All three trace tables + HNSW index in one database. Join between Avro, Vector, and Graph is a local SQL operation.
## Source: https://duckdb.org/docs/stable/guides/python/multiple_threads.html
