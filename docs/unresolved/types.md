# Type Questions

This document tracks questions left unresolved by the [types](../design/types.md#types-and-compile-time-knowledge) design. Statements here are not guarantees until their answers are incorporated into the design.

## Type-Provided Definitions

Should definitions inside types act as defaults or reusable capabilities beyond their current ability to satisfy declarations?

Their long-term role has not been specified.

## Invariant Solver

Which normalization, implication, and interpretation rules should the invariant solver implement?

The solver need not be complete, but its practical boundaries and treatment of complex mathematics remain open.

## Trusted Assumption Form

What exact operator form and syntax should expose trusted invariant assumptions?

`assuming Invariant` is the preferred direction. Any final form must reject disproven claims, accept proven claims normally, and make an unknown claim an explicit trusted boundary.

## Static Type Syntax

How should a contract state that a value is available during compilation without exposing its exact value?

`static` is the preferred term, but its place in type syntax is not finalized.
