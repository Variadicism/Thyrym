# Module Decisions

This document records the reasoning behind Thyrym's [modules](../design/modules.md#modules-and-imports) without extending their guarantees.

## Import Scoping

Imports expose the context established by an imported file but do not leak that file's imports transitively. This keeps dependencies explicit.

Treating paths as constrained static string values reuses normal expression machinery and permits compile-time path construction without a separate filename grammar.

Modeling `using` as an operator over a continuation makes imports locally scopeable. Normal operator-stack priority determines conflicts, so selective imports, narrower scope, or reordering can resolve them without a separate precedence system.
