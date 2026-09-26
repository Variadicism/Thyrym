# Type Decisions

This document records the reasoning behind Thyrym's [types](../design/types.md#types-and-compile-time-knowledge) without extending their guarantees.

## Structural Types And Variance

Types describe capabilities and invariants rather than nominal ancestry. Operator requirements use contravariant arguments and covariant returns because a satisfying implementation must accept every input promised by the requirement and may produce a more specific result than promised.

When repeated generic parameters match different argument types, combining matches with `or` preserves all possibilities instead of rejecting valid composition. A type error is deferred until code requires a capability the combined type cannot guarantee.

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
