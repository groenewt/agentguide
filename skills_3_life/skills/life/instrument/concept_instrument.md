# Concept: Instrument — A Measurement Tool
## Olog: Instrument → Life "measures", Instrument → Event "produces".
## AbstractInstrument with measure(), emit(). An instrument is a functor from C_Brain to C_Life.Event — it maps domain morphisms to observable events.
## The @traced decorator IS an instrument: it wraps a function, measures timing, and emits a Life.Event.
## Adjunction: Instrument ⊣ Forget (measuring wraps a mechanism; forgetting strips instrumentation).
## Sources: Mac Lane (1971) Ch.IV (adjunctions).
