# Core Decisions

This document records the rationale for foundational choices in Thyrym's [core](../design/core.md#overview). It explains the current design without extending its guarantees.

## Explicit Failure Instead Of Runtime Errors

Thyrym rejects invalid program states during compilation and represents legitimate external failures as values. This separates failures code must account for from language-level errors static checking should prevent.

Foreign contracts and assumed invariants remain trusted boundaries because incorrect information at either boundary can invalidate a proof.

## Pure Semantics With Unspecified Evaluation

Ordinary computation is referentially transparent, while observable interaction is represented by typed monadic acts. The language specifies results, effects, and dependencies rather than a strict or lazy schedule so implementations can evaluate during compilation, reorder independent work, parallelize it, or choose another semantics-preserving strategy.

## A Small Implementation-Guided Core

Many constructs that other languages build into their compilers are intended to use Thyrym's ordinary operator and type machinery. The final primitive inventory is therefore guided by implementation experience rather than fixed before bootstrapping reveals what is truly intrinsic.
