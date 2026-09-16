# Core Questions

This document tracks questions left unresolved by Thyrym's foundational [core](../design/core.md#thyrym-core-foundations). Statements here are not guarantees until their answers are incorporated into the design.

## Irreducible Core

Which operators, types, and compiler facilities must be intrinsic rather than implemented through ordinary operators?

Implementation must determine whether the likely elements listed in [core](../design/core.md#the-irreducible-core) are all necessary and whether any omitted facility cannot be bootstrapped.

## Additional Targets

Which trusted interface or primitive mechanisms will compilation targets other than C require?

The current design targets C and leaves target-specific additions for future work.
