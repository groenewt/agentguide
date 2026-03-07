# Tool: Bytecode Verification via dis

## Purpose
Confirm CALLED log is first EXECUTED instruction after decorator/metaclass transformations.

## Method
```python
import dis
instrs = list(dis.get_instructions(func))
# Skip RESUME, then find: LOAD_GLOBAL _log → LOAD_ATTR debug → LOAD_CONST "CALLED:..."
```

## Why Needed
AST catches structural presence. Decorators can reorder execution. Bytecode catches the actual instruction sequence.

## Sources
- https://docs.python.org/3/library/dis.html
- CPython bytecode spec: https://docs.python.org/3/library/dis.html#python-bytecode-instructions
