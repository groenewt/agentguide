# Concept: AbstractType — Root Type Classification

## Olog Position
Object "AbstractType" in C_Travel.Types. Morphism: AbstractType → Types "roots".

## Definition
AbstractType(AbstractThing, Enum, metaclass=ABCEnumMeta). The root of all categorical type classifications. Carries __type_meta__ and __algebra__ class variables. Each concrete type subclass is a finite set of classifiers (an enumeration), making Types a functor from C_Travel to FinSet (the category of finite sets).

## Why Both ABC and Enum
Types must be both declarative (ABC — abstract methods for type operations) and enumerative (Enum — finite, exhaustive classification). ABCEnumMeta resolves the metaclass conflict (see Resource.Common.Meta).

## Sources
- Spivak (2014). §3.4 — finite categories and enumerations.
