# Concept: 64-Bit Category Bitmask

## Definition
8 groups × 8 sub-categories = 64 bits (uint64_t). Each CALLED emission carries a bitmask identifying its rootfs position. The LogCategoryFilter uses bitwise AND to determine if an emission passes the current filter.

## Groups
| Bits | Group | rootfs | Pillar |
|---|---|---|---|
| 0–7 | SYSTEM | /sys,/proc | Resource |
| 8–15 | NETWORK | /dev | Communication |
| 16–23 | DATABASE | /var/lib | Location |
| 24–31 | SECURITY | /etc | Travel |
| 32–39 | UI | /usr | Sight |
| 40–47 | CORE | /lib,/bin | Resource |
| 48–55 | PERFORMANCE | /var/log | Life |
| 56–63 | INTEGRATION | /opt,/mnt | Location |

## Operations
mask_matches(event, filter) = (event & filter) ≠ 0
full_mask(group, sub) = group | sub

## Sources
- plan1.txt §Logging System — rootfs-Mapped Categories.
