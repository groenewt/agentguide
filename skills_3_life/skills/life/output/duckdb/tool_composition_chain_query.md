# Tool: Composition Chain Query
## Purpose: Reconstruct the morphism composition chain for a given arrow trace.
```sql
WITH RECURSIVE chain AS (
    SELECT from_seq, to_seq, 1 AS depth FROM trace_graph_edges
    WHERE to_seq = ? AND session_id = ? AND edge_type = 'CALLED_BY'
    UNION ALL
    SELECT e.from_seq, e.to_seq, c.depth + 1 FROM trace_graph_edges e
    JOIN chain c ON e.to_seq = c.from_seq
    WHERE e.session_id = ? AND e.edge_type = 'CALLED_BY' AND c.depth < 20
)
SELECT n.function_name, n.pillar, c.depth FROM chain c
JOIN trace_graph_nodes n ON n.session_id = ? AND n.sequence_id = c.from_seq
ORDER BY c.depth DESC;
```
