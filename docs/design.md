# Thyrym Design

The design documentation is divided by language concern:

- foundational goals and the language's central model are described in [core](design/core.md#thyrym-core-foundations);
- syntax, operators, contextual grammar, and recursion are described in [operators](design/operators.md#operators-and-grammar);
- structural types, invariants, primitive data, and compile-time knowledge are described in [types](design/types.md#types-and-compile-time-knowledge);
- acts, affine resources, and foreign interfaces are described in [effects](design/effects.md#effects-resources-and-trusted-interfaces);
- imports and compilation units are described in [modules](design/modules.md#modules-and-imports);
- monadic composition and contextual binding are described in [monads](design/monads.md#monads-and-contextual-binding);
- convergence, ongoing computation, stopping, awaiting, and races are described in [pursuits](design/pursuits.md#pursuits-convergence-and-stopping);
- scoped structural, generative, and probabilistic uniqueness is described in [uniqueness](design/uniqueness.md#scoped-uniqueness).

Decision explanations and unresolved questions are kept in the corresponding `rationale` and `unresolved` directories.
