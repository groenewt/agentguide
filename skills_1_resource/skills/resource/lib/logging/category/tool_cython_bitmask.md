# Tool: Cython Bitmask Compilation (_ops.pyx)

## File: resource/lib/logging/helpers/types/category/_ops.pyx

## Gospel XXI Criteria
1. Concrete: yes (pure arithmetic). 2. Called >1x/session: yes (every CALLED emission). 3. Statically typed: yes (uint64_t). 4. No dynamic dispatch: yes (bitwise only).

## Implementation
```cython
# cython: language_level=3, boundscheck=False, wraparound=False
from libc.stdint cimport uint64_t

cdef inline bint mask_matches(uint64_t event, uint64_t filter) nogil:
    return (event & filter) != 0

cpdef bint check_category(uint64_t event, uint64_t filter):
    return mask_matches(event, filter)

cpdef uint64_t full_mask(uint64_t group, uint64_t sub):
    return group | sub
```

## nogil Justification
Pure integer arithmetic. Releasing GIL allows Writer Cluster's three threads to run truly parallel.

## Fallback
```python
try:
    from ._ops import check_category, full_mask
except ImportError:
    from .manager import check_category, full_mask
```
