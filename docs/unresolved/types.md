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

## Quantifiable Return Types

What is the exact recursive definition of **quantifiable**, provisionally meaning a parameter or relation?

Open questions include whether a compound type is quantifiable when each variable component is quantifiable, whether concrete type-producing operators may wrap quantifiables, whether those operators must prove a relationship-preserving result, and how relation intersections compose.

The current direction is that return types should not silently introduce fresh generic structure, but this is not yet a finalized checking rule.

## Nested Relations

How can a relation select and preserve a nested part of another argument's type?

Important cases include the return context of an operator argument, an element role nested in a collection, the exact collection family of `Values`, and the return relation of a type-producing operator. `same as Values but Element := B` handles a directly exposed role but does not settle deeper projection.

## `a distinct ...`

What is the exact grammar and unification meaning of an intentionally independent type relation?

It remains open whether `distinct` creates a fresh parameter, forbids all unification, merely prevents an implicit link, or permits later unification when an explicit proof equates the types. Intentional independence rather than permanent disequality is the current direction.

## Implicit Structural Unification

Should repeated broad argument types share any relationship without an explicit parameter?

The design must determine whether shared roles can be discovered through common structural capabilities or whether argument types remain independent unless parameters connect them. Repeated source text alone must not be assumed to establish identity.

## Role Identity

What proves that nested roles in unrelated freeform types are semantically the same?

A promising direction is for a shared structural capability, such as a collection capability, to define its element role. A global convention based only on the spelling `Element` is not established.

## Partial Type Application

How should a freeform type-producing operator be partially applied when several named roles remain open?

`it` works for one missing role, but positional application conflicts with Thyrym's role-based design. Relations may reduce the need for partial type functions, but the remaining advanced use cases need syntax.

## `for any` Lists And Inline `any`

What final punctuation separates complex parameter declarations, and does inline `any` remain available?

The current preference uses `Name, a type`, allows semicolons between complex entries, and uses `=>` before an operator's return type. Line-separated entries remain possible. Front-loaded `for any` is clearer for complex signatures, but inline freshness may still have uses.

## Argument-Name Type Sugar

Should an argument whose name matches a parameter be allowed to omit its explicit type?

This may be useful local shorthand, but it must not encourage authors to create one parameter per argument and thereby lose meaningful shared relationships. It is not a substitute for the parameter and relation model.

## Relation Solver Integration

How do `same as`, explicit relation intersections, and `distinct` integrate with structural `or` combinations and the invariant solver's proven, disproven, and unknown outcomes?

Relations must use the existing semantic solver rather than create a parallel type system based on textual equivalence.
