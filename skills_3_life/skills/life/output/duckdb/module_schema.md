# Module: life/output/duckdb_schema.py — DDL for all trace tables.

## trace_avro: session_id UUID, sequence_id BIGINT, ts TIMESTAMP, source_module VARCHAR, pillar VARCHAR, category_mask UBIGINT, function_name VARCHAR, class_name VARCHAR, argument_summary VARCHAR, error_code VARCHAR. PK(session_id, sequence_id).

## trace_vectors: session_id UUID, sequence_id BIGINT, embedding FLOAT[384], function_name VARCHAR, category_mask UBIGINT. PK(session_id, sequence_id). HNSW index flash_idx WITH (metric='cosine').

## trace_graph_nodes: session_id UUID, sequence_id BIGINT, function_name VARCHAR, pillar VARCHAR, category_mask UBIGINT, depth INTEGER. PK(session_id, sequence_id).

## trace_graph_edges: session_id UUID, from_seq BIGINT, to_seq BIGINT, edge_type VARCHAR. PK(session_id, from_seq, to_seq, edge_type).

## trace_dead_letters: session_id UUID, sequence_id BIGINT, failed_trace VARCHAR, error_message VARCHAR, retry_count INTEGER, created_at TIMESTAMP.

## Sources
- DuckDB ARRAY: https://duckdb.org/docs/stable/sql/data_types/array
- DuckDB VSS: https://duckdb.org/docs/stable/core_extensions/vss
