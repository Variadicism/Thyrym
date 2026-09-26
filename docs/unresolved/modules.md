# Module Questions

This document tracks questions left unresolved by the [modules](../design/modules.md#modules-and-imports) design. Statements here are not guarantees until their answers are incorporated into the design.

## Visibility

How can several exported top-level operators share a helper without exporting it?

The design should prefer ordinary scope over explicit visibility modifiers if scope can solve the problem cleanly.

## Module Representation

How should separate compilation, compiled-module metadata, and exportable operator contexts be represented?

The current import model is provisional and specifies source-level behavior only approximately.

## Import Injection

What syntax and scope redirect an imported module or operator, and how does resolution identify the target across local imports, selective imports, and compiled modules?

Injection is intended as compile-time dependency replacement, particularly for tests, but it should remain visibly coupled to source organization rather than promise refactoring stability it cannot provide.
