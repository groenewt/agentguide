# Module: travel/thing.py
## Thing(@dataclass frozen=True) extending AbstractThing.
## Fields: uuid (UUID4), hash (int), created_at (datetime). All from Resource.Identity spec.
## Sacred methods: identify() returns Identity, describe() returns olog string, validate() checks fields, snapshot() returns frozenset of field values, constituents returns ().
## category="SECURITY" for all CALLED logs.
