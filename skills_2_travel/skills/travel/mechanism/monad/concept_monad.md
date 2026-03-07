# Concept: Monad — Computational Context (T, η, μ)

## Olog Position
Object "Monad" in C_Travel.Mechanism. Morphism: Monad → Mechanism "wraps".

## Definition
AbstractMonad with pure (η: Id ⇒ T), bind (Kleisli composition), fmap (T-action on morphisms). The Result type (Success | Failure) is the Either monad T(A) = A + E.

## Monad Laws (Must Be Verified in Tests)
1. Left unit: bind(pure(x), f) = f(x)
2. Right unit: bind(m, pure) = m
3. Associativity: bind(bind(m,f),g) = bind(m, λx.bind(f(x),g))

## Sources
- Mac Lane (1971). Ch.VI.
- Moggi (1991). "Notions of computation and monads." Info & Comp 93(1):55–92.
- Wadler (1995). "Monads for functional programming." AFP, Springer LNCS 925.
