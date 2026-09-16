# Effect Decisions

This document records the reasoning behind Thyrym's [effects](../design/effects.md#effects-resources-and-trusted-interfaces) without extending their guarantees.

## Precise Acts

Acts expose effects and dependencies through types instead of hiding all interaction behind an opaque IO category. The compiler can preserve required observable behavior while potentially reordering, parallelizing, eliminating, or duplicating acts when contracts prove those transformations safe.

## Affine Ownership

External resources need ownership semantics that prevent invalid duplication while allowing unused resources to be discarded. Affine typing fits that requirement and supplies information needed for effect-sensitive optimization and concurrency.

## Foreign Code As A Trusted Boundary

Thyrym cannot prove arbitrary foreign implementations safe or contract-compliant. Treating foreign declarations as explicit trusted boundaries preserves useful interoperability without weakening the meaning of guarantees made by safe Thyrym code.
