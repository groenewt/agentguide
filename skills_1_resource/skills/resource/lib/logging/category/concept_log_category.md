# Concept: LogCategory IntFlag

## Definition
A 64-bit IntFlag enum where each member is a power of 2. 8 parent groups, each with 7 sub-categories. CATEGORY_IMPLEMENTATION is the singleton manager that tracks _next_bit allocation.

## Verification
After all 64 categories allocated: CATEGORY_IMPLEMENTATION._next_bit == 64.
