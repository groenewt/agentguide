# Concept: ThingException — Failure IS a Thing

## Olog Position
Object "ThingException" in C_Res.Exception. Morphisms: ThingException → Exception "roots failure", ThingException → AbstractThing "failure IS a thing".

## Definition
ThingException(Exception, AbstractThing) — dual inheritance. Every failure in the system is itself a Thing: it has identity, can be described, validated, and snapshotted.

## Categorical Note
Dual inheritance claims ThingException is a pullback of Exception and AbstractThing over their common ancestor (object). The commutative diagram: (ThingException → Common → Resource) = (ThingException → Exception → Resource).

## Sources
- Mac Lane (1971). Ch.III §4 — pullbacks.
