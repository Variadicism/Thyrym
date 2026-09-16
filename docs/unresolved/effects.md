# Effect Questions

This document tracks questions left unresolved by the [effects](../design/effects.md#effects-resources-and-trusted-interfaces) design. Statements here are not guarantees until their answers are incorporated into the design.

## Act Contracts

How should act types describe dependencies, interference, duplication, elimination, reordering, and parallel execution precisely enough for static checking and optimization?

## Affine Resources

How should affine external resources, their lifetimes, and their consumption be represented in the type system?

The design may require a specialized built-in resource concept, but its exact representation is deferred.

## Foreign Contracts

What exact syntax and semantics should describe foreign inputs, outputs, failures, effects, resource mutation, ownership, and lifetimes?

Foreign declarations must remain explicit trusted boundaries because Thyrym cannot prove arbitrary implementations obey their contracts.
