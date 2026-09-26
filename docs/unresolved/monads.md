# Monad Questions

This document tracks details left unresolved by the [monads](../design/monads.md#monads-and-contextual-binding) design. Statements here are not language guarantees until their answers are incorporated into the design.

## Declaration Grammar

What exact Thyrym declarations define Functor, Applicative, Monad, transformer or distribution behavior, and the dangerous-discard property?

The architecture and semantic constraints are settled, but the surface grammar is not.

## Functor Mapping Syntax

Should `through` become the general-purpose mapping syntax, and how should it interact with `it`, nested expressions, and non-collection functors?

`through` is the current direction, not finalized core grammar.

## Nested Danger Detection

How is dangerous context detected through nested non-dangerous applicatives?

For example, after some binding depth leaves `a list of an error or strings`, the list itself is non-dangerous while its element context is dangerous. No blanket propagation rule has been selected.

## Explicit Sequencing Boundary

What syntax, scope, and semantics should force ordering between computations that have no data dependency but must execute in a particular order?

`wait` is a possible keyword, but remains provisional.

## Pattern Matching

What pattern-matching syntax should inspect and recover from maybe/perhaps, `an error or`, and other sum-like values retained as data?

## Implicit Solver

How should the general implicit conversion solver search and choose among outward lifts and declared lawful distributions?

The solver must never infer caret depth, invent distribution laws, unwrap contexts, or discard contextual information.

## Law Verification

How will Thyrym establish that user-defined monad, transformer, and distribution operations obey their required laws?

Possibilities include proof, testing, trusted declarations, or another mechanism.

## Preserving Nested Type Relations

How should a higher-order operation preserve the exact monad or effect specialization returned by its operator argument while changing a value nested inside it?

A map-like operation may need the exact task or act family from the mapper, the collection family from its input, and a new element type. Writing one concrete wrapper can erase a more specific monad subtype. This should be solved with the general parameter and relation model rather than ad hoc effect syntax.

## Diagnostic Wording

What exact diagnostics should explain an incorrect caret count or dangerous discard and trace a later type mismatch back to its binding?
