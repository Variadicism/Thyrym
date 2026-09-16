# Monad Decisions

This document records the reasoning behind the [monads](../monads.md#monads-and-contextual-binding) design without extending its guarantees.

## Natural Type Phrases

`an act making A` was chosen because `act` is concise without being an abbreviation and `making` reads naturally for both simple and nested result types. `an error or A` avoids an unnecessary `either` keyword, while `nil` names the absence of a useful successful value without importing the term `unit`.

Natural phrasing does not flatten type structure. Preserving constructor order is necessary because performing an act that may produce an error differs from already having either an error or an act.

## One Monad Concept With Optional Transformer Behavior

Monad and transformer behavior should be expressible by one conceptual type so users do not manage unrelated names for the same error or state semantics. Transformer behavior remains explicit because arbitrary monads do not compose lawfully. The naive list-transformer shape is a key counterexample.

Distribution is directional because effects cannot generally be inspected from pure code. An error around an act can move inside by constructing or mapping an act; an error inside an act cannot move outside without running that act.

## Conservative Implicit Conversion

Outward lifting is implicit because it adds a declared context without losing information. Declared lawful distribution is also meaning-preserving. Silent unwrapping, arbitrary normalization, or invented distribution could discard or reorder semantics, so those transformations are forbidden.

## Explicit Caret Depth

Caret depth is never inferred because different depths express different intentions. A caller may propagate an inner error or deliberately retain it for inspection and recovery. Expected types cannot determine that semantic choice safely.

The same rule applies to named binding and effect-only prefix sequencing, keeping a small, regular syntax.

## Applicative Inference Without New Assignment Syntax

A separate applicative pseudo-assignment operator was rejected because assignment-like syntax suggests that each name is immediately available to subsequent bindings. Dependency analysis under ordinary caret syntax preserves readability while allowing independent computations to compose applicatively.

Applicative inference decides dependency structure, not binding depth. Keeping those decisions separate lets the compiler optimize independent work without changing which contexts the programmer selected.

## Semantic Danger And Explicit Acknowledgement

Danger is attached to failure semantics rather than structural complexity. Errors are dangerous because discarding them hides failure; lists and optional absence are not dangerous merely because they can participate in contextual composition. An act is not dangerous solely because its successful payload is ignored.

The `!` acknowledgement was chosen instead of a family of discard symbols. Carets continue to express binding depth, while one orthogonal marker records intentional dangerous discard. This keeps unacknowledged failures invalid without forbidding deliberate best-effort operations.

Placing the danger property on Applicative avoids a parallel classification hierarchy and gives user-defined contextual types one ordinary place to opt into checking.

## Explicit Collection Of Effects

Lists do not distribute through arbitrary effects automatically because doing so would silently choose sequencing and collection semantics. Operations such as `combining errors` remain ordinary typed operations whose behavior is visible and independently definable.
