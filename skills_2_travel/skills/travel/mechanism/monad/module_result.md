# Module: travel/mechanism/monad/result.py
## Result = Success(value: T) | Failure(error: E). Concrete monad.
## pure(x) = Success(x). bind dispatches on is_success. fmap wraps f(value) in Success.
