# Module: resource/lib/logging/base.py
## Core stdlib import + LogCategoryLogger global setup.
## Calls logging.setLoggerClass(LogCategoryLogger) — GLOBAL side effect.
## After this, all logging.getLogger(__name__) calls return LogCategoryLogger instances.
