# Core Decisions

This document records the rationale for foundational choices in Thyrym's [core](../design/core.md#1-overview). It explains the current design without extending its guarantees.

## Explicit Failure Instead Of Runtime Errors

Thyrym rejects invalid program states during compilation and represents legitimate external failures as values. This separates failures that code must account for from language-level errors that static checking should prevent. Foreign contracts and assumed invariants remain trusted boundaries because incorrect information at either boundary can invalidate the proof.

## Pure Semantics With Unspecified Evaluation

Ordinary computation is referentially transparent, while observable interaction is represented by typed monadic acts. The language specifies results, effects, and dependencies rather than a strict or lazy schedule so implementations can evaluate at compilation, reorder independent work, parallelize it, or choose another semantics-preserving strategy.

## Operators As The Callable Abstraction

Using operators for all callable behavior avoids a separate function mechanism and lets ordinary computation, declarations, syntax extension, binding, and contextual constructs share one model. English-inspired grammar aims to make that flexibility readable without claiming that source code is literal English.

Pure operators generally use descriptive wording because they identify a resulting immutable value. Command-like wording is reserved primarily for acts, where it naturally describes an effect to perform.

## Newest Applicable Resolution

Resolving the newest applicable operator combines lexical shadowing with overloading. A newer syntactic match does not hide an older operator when its arguments, result, or invariants do not satisfy the use, so local specialization can coexist with broader definitions.

## Pre-Ops And Compile-Time Grammar

Pre-ops allow constructs to delay parsing and type checking until they have introduced the context needed by their bodies. This makes branch refinements, loop variables, bindings, and scoped domain-specific syntax instances of the same mechanism.

Generated grammar is restricted to compile-time-known information so every pre-op can be resolved before code is emitted. Runtime input cannot alter the grammar, preserving complete compilation and static checking.

## Structural Types And Variance

Types describe capabilities and invariants rather than nominal ancestry. Operator requirements use contravariant arguments and covariant returns because a satisfying implementation must accept every input promised by the requirement and may produce a more specific result than promised.

When repeated generic parameters match different argument types, combining the matches with `or` preserves all possibilities instead of rejecting otherwise valid composition. A type error is deferred until code requires a capability the combined type cannot guarantee.

## Trusted Assumptions

The assumption mechanism is intended for valid claims that a practical solver cannot prove. The compiler still rejects a claim it can disprove, accepts a claim it proves without trust, and treats only an unknown result as an explicit programmer assumption. This prevents the mechanism from overriding known falsehoods.

## Bits And A Small Core

`bits` is the primitive data-bearing type because every other representation ultimately needs stored data. Other concepts are built from bits, operators, types, and invariants to keep the irreducible language core small and allow ordinary language mechanisms to define constructs that other languages build into their compilers.

The final core is intentionally guided by implementation experience rather than fixed in advance, avoiding primitives that turn out to be expressible through the operator system.

## Static Values

`static` distinguishes complete compile-time availability from a contract that exposes a particular constant. This lets callers depend on compile-time computation without coupling them to an implementation's current value. Referential transparency then provides recursive propagation: a pure operator applied only to static arguments also has a static result.

## Precise Acts

Acts expose effects and dependencies through types rather than hiding all interaction behind an opaque IO category. The compiler can therefore preserve required observable behavior while potentially reordering, parallelizing, eliminating, or duplicating acts when their contracts prove those transformations safe.

## Import Scoping

Imports expose the context established by the imported file but do not leak that file's own imports transitively. This keeps dependencies explicit. Treating paths as constrained static string values reuses normal expression machinery and permits compile-time path construction without a separate filename grammar.

Modeling `using` as an operator over a continuation also makes imports locally scopeable. Normal operator-stack priority determines conflicts, so selective imports, narrower scope, or reordering can resolve them without a separate precedence system.

## Conventional And Deviant Syntax

The proposed `deviant` marker preserves unrestricted syntax while making the readable common dialect recognizable. Conventional forms could also support forward use because the compiler can recognize their shape before seeing a definition; unusual grammar would continue to require an explicit declaration.
