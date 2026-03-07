# Module: resource/common/base.py

## Implements
AbstractThing class with five sacred abstract methods.

## File Pattern
```python
"""AbstractThing — the primordial root of existence.
Olog: AbstractThing → Common "roots"
Pillar: Resource | rootfs: /sys (SYSTEM)
"""
from __future__ import annotations
import logging
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING
if TYPE_CHECKING:
    from typing import FrozenSet

_log = logging.getLogger(__name__)

class AbstractThing(ABC):
    @abstractmethod
    def identify(self): raise NotImplementedError
    @abstractmethod
    def describe(self) -> str: raise NotImplementedError
    @abstractmethod
    def validate(self) -> bool: raise NotImplementedError
    @abstractmethod
    def snapshot(self) -> FrozenSet: raise NotImplementedError
    @property
    @abstractmethod
    def constituents(self) -> tuple: raise NotImplementedError

__all__ = ["AbstractThing"]
```

## Gospel Compliance
- XIV: No function bodies (abstract declarations only). ✓
- XV: All methods take 0 params (excl self). ✓
- XVI: Abstract methods are EXCLUDED from CALLED log. ✓
- XVIII: No cross-pillar imports. ✓
- XIX: All methods declared in olog. ✓
