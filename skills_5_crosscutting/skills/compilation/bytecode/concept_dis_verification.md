# Concept: Bytecode Verification (dis)
## Confirms CALLED log is first EXECUTED instruction after decorators.
## Pattern: LOAD_GLOBAL _log -> LOAD_ATTR debug -> LOAD_CONST "CALLED:..."
## Source: docs.python.org/3/library/dis.html
