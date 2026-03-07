# Tool: Cython Cosine Similarity (_distance.pyx)
## Gospel XXI: concrete, called on flash reads, float[:] typed, no dispatch.
## cosine_similarity(float[:] a, float[:] b) → float. 384-dim loop = straight C.
## boundscheck=False, wraparound=False. ~10-50x faster than numpy for single pairs.
## Source: Cython documentation, https://cython.readthedocs.io/en/latest/src/userguide/numpy_tutorial.html
