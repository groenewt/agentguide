# Concept: Logging as Representable Functor

## Olog Position
Lib.Logging is the sub-category of Lib that implements Hom(−, Life): the representable functor whose image is the Observation Product.

## Categorical Justification
By the Yoneda lemma (Yoneda 1954, Mac Lane 1971 Ch.III §2), Nat(Hom(A,−), F) ≅ F(A). The logging system implements Hom(−, Life): for each module, it records all observable morphisms to the Life pillar. The Yoneda embedding guarantees this is fully faithful — log emission patterns reconstruct the morphism structure.

## Sources
- Yoneda, N. (1954). J. Fac. Sci. Univ. Tokyo 7: 193–227.
- Mac Lane (1971). Ch.III §2.
