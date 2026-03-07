# Concept: Lib — The Stdlib Functor

## Olog Position
Object "Lib" in C_Res. Morphism: Lib → Resource "wraps".

## Definition
Lib is a functor from the external category C_StdLib (Python's standard library) to C_Resource. It maps stdlib types to PyBrain-namespaced wrappers.

## Demand-Driven Rule
Lib wraps ONLY what other ologs import. No preemptive wrapping. At Step 2: abc, enum, typing, uuid, dataclasses. Additional modules added ONLY when a pillar's olog requires them.

## Why Not a Package Per Module
Single-file wrappers (lib/abc.py, lib/enum.py) unless the olog has internal objects requiring decomposition. Only lib/logging/ is a full package because its olog declares LogCategoryLogger, LogCategoryFilter, etc. as distinct objects with internal morphisms.

## Sources
- Functorial wrapping: Spivak (2014) §1.2 (functors as structure-preserving translations).
