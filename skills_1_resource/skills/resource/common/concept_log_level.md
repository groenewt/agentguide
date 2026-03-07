# Concept: LogLevel — Severity of Observation

## Olog Position
Object "LogLevel" in C_Res.Common. Morphism: LogLevel → Common "classifies severity".

## Definition
IntEnum: DEBUG=10, INFO=20, AUDIT=25, WARNING=30, ERROR=40, CRITICAL=50. Maps directly to Python stdlib logging levels with AUDIT added for governance traces.

## Categorical Justification
LogLevel is a totally ordered set (a thin category where Hom(a,b) has at most one element iff a ≤ b). The ordering determines which morphism traces pass through the observation filter.
