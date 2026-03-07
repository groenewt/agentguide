# Concept: Language — Serialization Layer

## Olog Position
Object "Language" in C_Travel. Morphism: Language → Travel "serializes".

## Definition
AbstractLanguage(AbstractThing) with encode/decode. Language is the functor that maps domain objects to wire representations. Each sub-object (Grammar, Lexicon, Semantics, Serde, Format, Translator) handles a different aspect of the serialization/deserialization pipeline.

## Sub-Objects
Grammar "parses" (syntax rules), Lexicon "looks up" (vocabulary), Semantics "interprets" (meaning), Serde "serializes/deserializes", Format "to_wire/from_wire", Translator "translates" (between formats).
