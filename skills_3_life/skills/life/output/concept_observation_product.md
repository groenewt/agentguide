# Concept: Observation Product — The Triple-Trace Functor

## Olog: Output → Life "records". Output is WHERE the Observation Product writes.

## Formal Definition
The Observation Product is the product functor:
  Observe: C_Brain → C_Avro × C_Vec × C_Graph
with projections π₁ (Avro), π₂ (Vector), π₃ (Graph).

Every CALLED emission f in C_Brain maps to:
  Observe(f) = (avro(f), vec(f), graph(f))

The shared key (session_id, sequence_id) is the identity that makes all three projections joinable.

## Categorical Properties
- Faithful: distinct arrows → distinct traces (Yoneda lemma, Mac Lane 1971 Ch.III §2).
- Product: the triple IS the categorical product in the observation target category.
- Fault-tolerant: 2-of-3 projections suffice for approximate reconstruction.

## Sources
- Mac Lane (1971). Ch.III §2 (Yoneda), Ch.III §4 (products).
- DuckDB multi-thread: https://duckdb.org/docs/stable/guides/python/multiple_threads.html
