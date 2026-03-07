# Concept: ABCEnumMeta — The Metaclass Bridge

## Olog Position
Object "Meta" in C_Res.Common. Morphism: Meta → Common "bridges ABC+Enum".

## Definition
A metaclass that resolves the metaclass conflict between ABCMeta (for abstract base classes) and EnumMeta (for enumerations). Required because Travel.Types.AbstractType must be both abstract (declare morphisms) and enumerable (classify instances).

## Categorical Justification
ABCMeta and EnumMeta are both endofunctors on the category of Python types. ABCEnumMeta is their pushout — the universal metaclass that admits both ABC and Enum structure. In categorical terms: ABCEnumMeta = ABCMeta ⊔_{type} EnumMeta.
