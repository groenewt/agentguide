# Module: life/output/connection.py
## ClusterConnection: manages DuckDB .cursor() per worker_id. cursor_for(CursorRequest) → DuckDBPyConnection. Enforces Gospel XXIII: no ad-hoc threads, all concurrency through declared cluster.
