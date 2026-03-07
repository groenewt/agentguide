# Concept: Composition as Kleisli Category

## Olog Position
Object "Composition" in C_Travel.Mechanism. Morphisms: Composition → Mechanism "composes", Composition → Monad "is Kleisli category of".

## Definition
AbstractComposition with pipe, compose, identity. These ARE Kleisli composition: g ∘_T f = μ_B ∘ T(g) ∘ f. pipe(f, g) = compose(g, f). identity = pure.

## Sources
- Mac Lane (1971). Ch.VI §5 (Kleisli category).
