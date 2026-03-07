# Tool: Flash Context Query
## Purpose: "What has the brain done recently that is similar to what I am about to do?"
```sql
SELECT a.function_name, a.pillar, a.ts
FROM trace_vectors v
JOIN trace_avro a ON v.session_id = a.session_id AND v.sequence_id = a.sequence_id
ORDER BY array_cosine_similarity(v.embedding, ?::FLOAT[384]) DESC
LIMIT 5;
```
