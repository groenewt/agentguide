# Concept: Thing — A Concrete, Frozen, Identified Entity

## Olog Position
Object "Thing" in C_Travel. Morphism: Thing → Travel "travels as". Thing → AbstractThing "extends" (via Resource functor).

## Definition
@dataclass(frozen=True). Carries UUID, hash, created_at. The concrete realization of AbstractThing — while AbstractThing is the declaration (morphism schema), Thing is the instance (element of Set under the instance functor I: C_Travel → Set).

## Frozenness
frozen=True makes Thing an object in the full subcategory of immutable types. This is required by Gospel XV: product objects must be frozen.

## Sources
- Spivak & Kent (2012). §2.3 — instances of ologs.
