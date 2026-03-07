# Concept: Rule of Threes — Consensus, Fault Tolerance, Parallelism

## Structural Isomorphism
| CPU | Observation Product | Vector Tiers | Distributed Systems |
|---|---|---|---|
| L1 cache | Avro (fast serial) | Flash (384-dim) | Primary replica |
| L2 cache | Graph (structural) | Reference (768-dim) | Secondary replica |
| L3 cache | Vector (searchable) | Memory (1024-dim) | Tertiary replica |

Three provides: consensus (all agree on identity), fault tolerance (2-of-3 survival), parallelism (distinct access patterns: serial replay, structural query, similarity search).

## Filtration of Forgetful Functors
Memory → Reference → Flash is a chain of forgetful functors, each discarding embedding dimensions. The left adjoints (embed at each tier) are the best approximations each tier can provide.

## Sources
- CPU cache hierarchy analogy: Patterson & Hennessy (2017). *Computer Organization and Design*. §5.1.
- Consensus: Lamport (1998). "The Part-Time Parliament." ACM TOCS 16(2).
