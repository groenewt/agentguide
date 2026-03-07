# Concept: Factory — Dependency Injection

## Olog Position
Object "Factory" in C_Travel. Morphism: Factory → Travel "injects".

## Definition
AbstractFactory with create, register, resolve. Factory is the LEFT ADJOINT of the forgetful functor from generated objects back to specifications: Factory ⊣ Forget. create() is the unit η of this adjunction.

## Sources
- Mac Lane (1971). Ch.IV (adjunctions).
- Gamma et al. (1994). *Design Patterns*. Addison-Wesley. (Factory pattern — but categorically grounded here.)
