# Type Decisions

This document records the reasoning behind Thyrym's [types](../design/types.md#types-and-compile-time-knowledge) without extending their guarantees.

## Structural Types And Variance

Types describe capabilities and invariants rather than nominal ancestry. Operator requirements use contravariant arguments and covariant returns because a satisfying implementation must accept every input promised by the requirement and may produce a more specific result than promised.

When repeated generic parameters match different argument types, combining matches with `or` preserves all possibilities instead of rejecting valid composition. A type error is deferred until code requires a capability the combined type cannot guarantee.

## Parameters Are Links

Because every type is structural and extensible, repeated text such as `a float` does not identify one concrete specialization. A named parameter is therefore meaningful rather than boilerplate: it links occurrences and carries their jointly inferred capabilities into the result.

Combining actual types with structural `or` and describing the usable result as their capability intersection are compatible views. The alternatives are retained, while only capabilities common to every alternative are guaranteed.

`for any` is preferred to `for all` because it reads naturally with the singular appositional form `F, a float`. Parameters may remain unconstrained when an algorithm does not semantically require a narrower capability, avoiding accidental over-specialization.

## Directional Relations

A shared parameter is symmetric: every occurrence contributes to one inferred common type. Some results are instead determined by one argument. `same as Left` records that direction without forcing unrelated arguments to share `Left`'s representation.

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

The assumption mechanism supports valid claims that a practical solver cannot prove. The compiler rejects a claim it disproves, accepts a claim it proves without trust, and treats only an unknown result as a programmer assumption. It cannot override a known falsehood.

## Logical Exclusion Instead Of XOR Types

Exclusive alternatives refine what states are possible, so they belong in invariants. Structural types describe provided components and capabilities and generally cannot promise the absence of additional capabilities. Keeping XOR out of structural composition avoids implying an exclusion that structural satisfaction cannot preserve.

## Bits And A Small Core

`bits` is primitive because every data representation ultimately needs stored data. Other concepts are built from bits, operators, types, and invariants, allowing the irreducible core to stay small.

The final core is guided by implementation experience rather than fixed in advance, avoiding primitives that prove expressible through the operator system.

## Static Values

`static` distinguishes complete compile-time availability from a contract exposing a particular constant. Callers can depend on compile-time computation without coupling to an implementation's current value.

Referential transparency gives natural propagation: a pure operator applied only to static arguments has a static result.
