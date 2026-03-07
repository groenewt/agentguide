# Concept: Identity — The Unique Essence

## Olog Position
Object "Identity" in C_Res.Common. Morphism: Identity → Common "identifies".

## Definition
An enum specifying the fields that uniquely identify any AbstractThing instance: UUID, HASH, CREATED_AT, TYPE. Each variant carries a field_type and default_factory.

## Categorical Justification
In Set, identity of an element is determined by its membership. In a typed category, identity requires a typing discipline — the Identity enum specifies what constitutes sameness. This addresses the identity conditions deficit described in the GraphAtlas paper §2.2.1.

## Sources
- Spivak & Kent (2012). §2.1 — identity conditions for olog objects.
- GraphAtlas OLOG.md §2.2.1 — identity conditions in knowledge graphs.
