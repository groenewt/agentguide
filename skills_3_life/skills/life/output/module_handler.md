# Module: life/output/handler.py
## ObservationProductHandler(logging.Handler). Spawns three Writer Cluster workers per CALLED emission. Assigns sequence_id via AtomicCounter before fan-out. Reports to DuckDBOutput for actual inserts.
