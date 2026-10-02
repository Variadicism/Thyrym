# Type Decisions

This document records the reasoning behind Thyrym's [types](../design/types.md#types-and-compile-time-knowledge) without extending their guarantees.

## Structural Types And Variance

Types describe capabilities and invariants rather than nominal ancestry. Operator requirements use contravariant arguments and covariant returns because a satisfying implementation must accept every input promised by the requirement and may produce a more specific result than promised.

When repeated generic parameters match different argument types, combining matches with `or` preserves all possibilities instead of rejecting valid composition. A type error is deferred until code requires a capability the combined type cannot guarantee.

Extensionally structural identity keeps membership open: capabilities supplied by another library or local context can establish a type without modifying the value's original declaration. An explicit satisfaction declaration remains useful documentation but is a compiler-checked promise rather than nominal ownership of membership.

The declaring type is excluded from property identity so structurally compatible requirements can share implementations. A nominal escape hatch remains deferred instead of weakening ordinary structural semantics.

## Properties And Evidence

Property-ness belongs to a type requirement because dynamic dispatch is a promise made by the exposed interface. Marking implementations specially would duplicate operator mechanisms and prevent one ordinary definition from serving direct calls and one or more compatible properties.

Selecting property implementations when type satisfaction is established preserves abstraction across lexical boundaries. Distant generic code needs only the carried evidence, while the raw value can acquire a different valid typed view in another context without changing globally.

Dynamic dispatch through exposed evidence does not imply arbitrary reflection. Parametric code may observe precisely the capabilities promised by its interface while hidden representation and unrelated capabilities remain abstract.

## Implicit Existential Abstraction

Heterogeneous structural values require existential abstraction conceptually, but visible `Some*` wrappers would impose boilerplate without adding semantics. Leaving representation to the compiler permits dictionaries, vtables, tags, specialization, or direct calls while source code simply uses the common structural type.

## Parameters Are Links

Because every type is structural and extensible, repeated text such as `a float` does not identify one concrete specialization. A named parameter is therefore meaningful rather than boilerplate: it links occurrences and carries their jointly inferred capabilities into the result.

The exact structural `or` of actual bindings is retained. It exposes common capabilities without discarding which alternatives established them. Automatically replacing `dog or cat` with `pet` would lose information; the user can choose the broader view explicitly when desired.

`for any` is preferred to `for all` because it reads naturally with the singular appositional form `F, a float`. Parameters may remain unconstrained when an algorithm does not semantically require a narrower capability, avoiding accidental over-specialization.

## Directional Relations

A shared parameter is symmetric: every occurrence contributes to one exact accumulated `or` type. Some results are instead determined by one argument. `same as Left` records that direction without forcing unrelated arguments to share `Left`'s representation.

`same as Values but Element := B` preserves readable structural information and changes only the role the operation actually changes. Requiring a separate parameter for every list, function, or wrapper would produce opaque signatures and can impose equality that the implementation does not need.

Intentional independence also needs to be visible. The provisional `a distinct ...` relation prevents an author from accidentally communicating preservation when the implementation may choose another satisfying type. **Distinct** avoids the allocation connotations of **new**.

## Return Information Must Not Disappear

Return declarations deserve stricter scrutiny because restating a broad type can silently discard specialization carried by arguments. Parameters and relations are the current foundation for retaining that information while leaving ordinary argument descriptions readable.

The broader **quantifiable** rule remains provisional because nested type-producing operators and monads must preserve relationships compositionally. A concrete wrapper must not erase a more specific contextual type such as a task merely because both satisfy a broader act or IO-like capability.

## Capability-Based Portability

Shader portability follows from ordinary structural dispatch rather than duplicate shader primitives or a macro language. Float values perform arithmetic, while GLSL float-expression values can implement the same operators by constructing shader expressions. Generic functions remain ordinary Thyrym code.

Environment names such as CPU and GPU are poor semantic type boundaries because available capabilities can change independently of deployment. General float, width-specific, and IEEE-specific capabilities state what an implementation actually supports.

## Named Roles Instead Of Positional Families

Positional type-function application makes future abstraction depend on arbitrary parameter order. Thyrym's named, role-based operator model should apply to type-producing operators as well. The cost of multiple open roles should be addressed explicitly rather than by importing positional assumptions.

## Trusted Assumptions

The assumption mechanism supports valid claims that a practical solver cannot prove. The compiler rejects a claim when any admitted case is disproven, accepts a claim it proves without trust, and treats only an entirely unknown remainder as a programmer assumption. It cannot override a known falsehood or cast away a known alternative.

Explicit checked refinement is separate from trust. A runtime condition can divide execution into typed outcomes and refine the successful branch without pretending the proposition was statically proven or turning failure into a hidden language error.

Provably contradictory invariants are rejected because they describe no valid value. If deliberate emptiness becomes useful, an explicit empty type communicates that intent better than contradictory requirements.

## Invariants And Laws

Invariants describe particular values, while laws supply universal knowledge to the solver. Keeping both in the ordinary proof system lets declaration guarantees compose from lower-level contracts without requiring a separate refinement hierarchy.

Universal laws often cannot be derived from source implementations, so programmer-provided laws use the assumption boundary and remain invalid when any admitted counterexample is known. They are compile-time solver knowledge rather than runtime exhaustive checks.

## Unambiguous Conversion And Minimal Inference

Rejecting ambiguous implicit conversions prevents hidden compiler preference and circular inference. Requiring an explicit counterpart for every implicit conversion keeps the programmer able to resolve ambiguity visibly.

Minimal inference preserves exact information: definitions can receive the precise type of their body instead of an automatically widened approximation, while declarations and arguments state the contracts needed for separate checking and resolution.

## Relational Capabilities

Capabilities such as `equatable to T` express operand relationships structurally without recovering hidden nominal identity. Heterogeneous storage alone cannot promise that every pair supports the relationship, so sorting or equality must require sufficient exposed evidence rather than invent fallback behavior for incomparable values.

## Logical Exclusion Instead Of XOR Types

Exclusive alternatives refine what states are possible, so they belong in invariants. Structural types describe provided components and capabilities and generally cannot promise the absence of additional capabilities. Keeping XOR out of structural composition avoids implying an exclusion that structural satisfaction cannot preserve.

## Bits And A Small Core

`bits` is primitive because every data representation ultimately needs stored data. Other concepts are built from bits, operators, types, and invariants, allowing the irreducible core to stay small.

The final core is guided by implementation experience rather than fixed in advance, avoiding primitives that prove expressible through the operator system.

## Static Values

`static` distinguishes complete compile-time availability from a contract exposing a particular constant. Callers can depend on compile-time computation without coupling to an implementation's current value.

Referential transparency gives natural propagation: a pure operator applied only to static arguments has a static result.
