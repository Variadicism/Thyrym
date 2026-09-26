# Module Decisions

This document records the reasoning behind Thyrym's [modules](../design/modules.md#modules-and-imports) without extending their guarantees.

## Import Scoping

Imports expose the context established by an imported file but do not leak that file's imports transitively. This keeps dependencies explicit.

Treating paths as constrained static string values reuses normal expression machinery and permits compile-time path construction without a separate filename grammar.

Modeling `using` as an operator over a continuation makes imports locally scopeable. Normal operator-stack priority determines conflicts, so selective imports, narrower scope, or reordering can resolve them without a separate precedence system.

## Import Injection

Import injection provides general compile-time dependency replacement and is useful for tests, but it necessarily refers to source or module organization. Refactoring that organization can therefore break an injection without changing behavior.

It is not used as the sole redirection mechanism because captured affine resources have a more structurally stable substitution point. Capture injection can replace the resource while preserving its promised capabilities regardless of the library path used to reach it.
