# Tool: Cluster Integration Tests
## test_triple_trace.py: single CALLED → 3 DuckDB inserts (verify all 3 tables populated).
## test_cluster_degradation.py: kill vector_writer mid-session → verify avro+graph continue, dead letter created.
## test_tier_cascade.py: inject low-confidence flash result → verify escalation to reference.
## test_dead_letter_replay.py: create dead letters → trigger replay → verify trace_vectors populated.
