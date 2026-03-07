# Tool: Sacred Method Verification

## Purpose
Verify that AbstractThing subclasses implement all five sacred methods, and that each concrete implementation carries a CALLED log.

## Method
1. Use inspect.isabstract() to confirm AbstractThing is abstract.
2. For each concrete subclass, verify identify/describe/validate/snapshot/constituents exist.
3. For each concrete method, verify first bytecode instruction is _log.debug("CALLED:...").

## File: app/tests/unit/resource/test_abstract_thing.py
