# Type Questions

This document tracks questions left unresolved by the [types](../design/types.md#types-and-compile-time-knowledge) design. Statements here are not guarantees until their answers are incorporated into the design.

## Type-Provided Definitions

Should definitions inside types act as defaults or reusable capabilities beyond their current ability to satisfy declarations?

Their long-term role has not been specified.

## Invariant Solver

Which normalization, implication, and interpretation rules should the invariant solver implement?

The solver need not be complete, but its practical boundaries and treatment of complex mathematics remain open.

## Property Arity

Must every property requirement contain exactly one `one` argument?

Exactly one instance position would make the owner of dynamic dispatch unambiguous for binary relationships such as equality, but the rule has not been finalized.

## Hidden Relational Capabilities

How can independently abstracted structural values recover a relationship that depends on compatible hidden types?

If cats are equatable to cats and dogs to dogs, two values exposed only as pets need to distinguish cat/cat, dog/dog, and cat/dog without accidentally granting unrestricted structural reflection. Possible relational evidence models remain unresolved.

## Runtime Structural Reflection

May code test for a capability absent from a value's exposed static type, such as asking whether a value known only as `named` is also `messageable`?

The current direction favors parametricity and avoiding unrestricted reflection, but hidden relational capabilities may require a carefully bounded mechanism.

## Runtime Discrimination Of `or`

Does possession of `A or B` permit runtime code to determine whether the current value satisfies `A`, `B`, or both?

`or` is settled as inclusive, but conventional tagged-union elimination semantics are not established.

## Nominal Escape Hatch

Should Thyrym provide a compiler or native mechanism for deliberately distinguishing types with identical capabilities and invariants?

Ordinary type identity remains structural. No nominal mechanism has been designed.

## Recursive Structural Satisfaction

What sound termination or cycle rule governs recursive capability proofs?

Existing compiler machinery may already answer part of this question and should be reviewed before a replacement is designed.

## Explicit Empty Type

Does Thyrym need an intentionally uninhabited type, and if so, what explicit representation should it use?

Provably contradictory invariants are errors rather than an implicit empty-type encoding.

## Trusted Assumption Form

What exact operator form and syntax should expose trusted invariant assumptions?

`assuming Invariant` is the preferred direction. Any final form must reject disproven claims, accept proven claims normally, and make an unknown claim an explicit trusted boundary.

## Checked Refinement Syntax

What explicit operator or control-flow syntax tests a runtime-dependent invariant and refines only its successful outcome?

An unknown obligation must not become an automatic failing assertion. Failure needs a typed branch or result consistent with Thyrym's no-hidden-runtime-error rule.

## Universal Law Syntax

What exact syntax declares user-provided universally quantified solver laws?

`assuming for all ...` is the current conceptual form, and no separate `law` keyword is presently required. Any final form must reject a law when an admitted counterexample is proven.

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
