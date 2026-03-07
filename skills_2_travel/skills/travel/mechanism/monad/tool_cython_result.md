# Tool: Cython Result Monad (_result.pyx)

## Gospel XXI Criteria Met
1. Concrete. 2. Called on every Result chain. 3. bint + object statically typed. 4. No dynamic dispatch in bind/fmap.

## CResult cdef class
cdef bint _is_success, cdef object _value. cpdef bind/fmap/pure/fail. Must pass identical monad law tests as pure-Python Result.
