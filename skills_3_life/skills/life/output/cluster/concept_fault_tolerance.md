# Concept: 2-of-3 Fault Tolerance
## avro_writer: REQUIRED (replay source). graph_writer: REQUIRED (composition integrity). vector_writer: OPTIONAL (rebuildable via re-embedding from avro text).
## Dead-letter table: trace_dead_letters. Retry: max 3, backoff [100ms, 500ms, 2000ms].
## Background replay thread re-embeds from function_name + source_module on vector recovery.
