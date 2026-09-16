# Core Questions

This document tracks questions left unresolved by the foundational Thyrym [core](../design/core.md#37-open-design-questions). Statements in this document are not language guarantees until their answers are incorporated into the design.

## Operators And Bootstrapping

### Irreducible Core

Which operators, types, and compiler facilities must be intrinsic rather than implemented through ordinary operators?

Implementation work must determine whether the likely core elements listed in [core](../design/core.md#35-the-irreducible-core) are all necessary and whether any omitted facility cannot be bootstrapped.

### Conventional Operator Forms

Which grammatical shapes should count as conventional, non-`deviant` operator forms?

The forms must be constrained enough for the compiler to recognize forward uses reliably while still covering ordinary operator syntax. The corresponding forward-recognition algorithm also remains to be designed.

### Completing Forward Declarations

How should an implementation be attached to an existing operator declaration?

The mechanism should refer to the declaration's identity without requiring a complex operator pattern to be repeated.

### Type-Provided Definitions

Should definitions inside types act as defaults or reusable capabilities beyond their current ability to satisfy declarations?

Their long-term role has not been specified.

## Type Knowledge

### Invariant Solver

Which normalization, implication, and interpretation rules should the invariant solver implement?

The solver is intentionally not required to be a complete theorem prover, but its practical boundaries and handling of complex mathematics remain open.

### Trusted Assumption Form

What exact operator form and syntax should expose trusted invariant assumptions?

`assuming Invariant` is the preferred direction. Any final form must reject disproven claims, accept proven claims normally, and make an unknown claim an explicit trusted boundary.

### Static Type Syntax

How should a contract state that a value is available during compilation without exposing its exact value?

`static` is the preferred term, but its place in type syntax has not been finalized.

## Effects And Resources

### Act Contracts

How should act types describe dependencies, interference, duplication, elimination, reordering, and parallel execution precisely enough for static checking and optimization?

### Affine Resources

How should affine external resources, their lifetimes, and their consumption be represented in the type system?

The design may require a specialized built-in resource concept, but its exact representation is deferred.

## Imports And Modules

### Visibility

How can several exported top-level operators share a helper without exporting that helper?

The design should prefer ordinary scope over explicit visibility modifiers if scope can solve the problem cleanly.

### Module Representation

How should separate compilation, compiled-module metadata, and exportable operator contexts be represented?

The current import model is provisional and specifies source-level behavior only approximately.

## Trusted Interfaces

### Foreign Contracts

What exact syntax and semantics should describe foreign inputs, outputs, failures, effects, resource mutation, ownership, and lifetimes?

Foreign declarations must remain an explicit trusted boundary because Thyrym cannot prove that arbitrary foreign implementations obey their contracts.

## Compilation Details

### Additional Targets

Which trusted interface mechanisms will compilation targets other than C require?

The current design targets C and leaves target-specific additions for future work.
