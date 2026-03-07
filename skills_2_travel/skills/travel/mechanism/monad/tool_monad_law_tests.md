# Tool: Monad Law Verification

## Tests (app/tests/unit/travel/test_monad_laws.py)
```python
def test_left_unit():
    f = lambda x: Result.pure(x + 1)
    assert Result.pure(42).bind(f) == f(42)

def test_right_unit():
    m = Result.pure(42)
    assert m.bind(Result.pure) == m

def test_associativity():
    f = lambda x: Result.pure(x + 1)
    g = lambda x: Result.pure(x * 2)
    m = Result.pure(10)
    assert m.bind(f).bind(g) == m.bind(lambda x: f(x).bind(g))
```
Both CResult (Cython) and Result (Python) must pass identically.
