# Module: life/instrument/traced.py
## @traced decorator. Wraps a function with CALLED log + timing measurement.
## Emits a Life.Event with duration_ms, function_name, category_mask.
## This is the concrete instrument that every CALLED emission passes through.
