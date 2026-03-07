# Module: life/output/cluster.py
## WriterCluster class. Manages 3 writer threads via ClusterConnection (DuckDB cursor pool). AtomicCounter for sequence_id. Health reporting. Dead-letter dispatch on failure.
