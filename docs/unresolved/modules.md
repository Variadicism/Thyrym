# Module Questions

This document tracks questions left unresolved by the [modules](../design/modules.md#modules-and-imports) design. Statements here are not guarantees until their answers are incorporated into the design.

## Visibility

How can several exported top-level operators share a helper without exporting it?

The design should prefer ordinary scope over explicit visibility modifiers if scope can solve the problem cleanly.

## Module Representation

How should separate compilation, compiled-module metadata, and exportable operator contexts be represented?

The current import model is provisional and specifies source-level behavior only approximately.
