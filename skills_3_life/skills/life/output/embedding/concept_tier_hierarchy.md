# Concept: Three-Tier Embedding Hierarchy

## Tiers
| Tier | Model | Dim | Params | Latency | Purpose |
|---|---|---|---|---|---|
| Flash | granite-embedding-small-english-r2 | 384 | 47M | <1ms | Immediate context |
| Reference | granite-embedding-english-r2 | 768 | 149M | ~5ms | Mid-resolution |
| Memory | TBD (high-dim) | 1024+ | 300M+ | ~20ms | Full semantic |

## Categorical Reading
Memory → Reference → Flash is a chain of forgetful functors (each discards dimensions).
The left adjoint at each level (embed at that tier) is the best approximation that tier provides.

## Sources
- granite-small-r2: https://huggingface.co/ibm-granite/granite-embedding-small-english-r2
- granite-r2: https://huggingface.co/ibm-granite/granite-embedding-english-r2
- Awasthy et al. (2025). "Granite Embedding Models." arXiv:2502.20204.
- Awasthy et al. (2025). "Granite Embedding R2 Models." arXiv:2508.21085.
