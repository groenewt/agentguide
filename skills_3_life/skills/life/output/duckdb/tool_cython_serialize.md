# Tool: Cython Trace Serialization (_serialize.pyx)
## Gospel XXI: concrete, called every emission, statically typed, no dynamic dispatch.
## pack_avro_record(...) → tuple ready for INSERT. pack_graph_edge(...) → tuple ready for INSERT.
## Eliminates Python tuple construction overhead on the hot path.
