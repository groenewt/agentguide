# Concept: Writer Cluster — Sequence Consensus (N=3)
## Topology: Star with fixed center (AtomicCounter). Not Raft. Not Paxos.
## Leader assigns sequence_id BEFORE fan-out to [Avro, Vector, Graph].
## Writers coordinate only with leader, not with each other.
## Protocol: 1) CALLED arrives → 2) seq = counter.increment() → 3) dispatch 3 writers → 4) each writer uses own DuckDB cursor → 5) health report.
## Source: Lamport, L. (2001). *Paxos Made Simple*. (Why we DON'T need this — single process, no distributed consensus.)
