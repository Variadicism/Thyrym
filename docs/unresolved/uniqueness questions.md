# Uniqueness Questions

This document tracks details left unresolved by the scoped [uniqueness](../uniqueness.md#scoped-uniqueness) design. Statements here are not language guarantees until their answers are incorporated into the design.

## Surface Declarations

What exact Thyrym declarations should expose structural unique lookup or filtering, probabilistic scope bounds, canonical full representations, and partial identifiers?

The syntax must follow Thyrym's operator, capability, invariant, and naming conventions rather than introduce conventional trait or method notation.

## Generator Metadata

What is the final trusted metadata schema for probabilistic generators?

In particular, is a conservative global distribution factor useful when effective collision behavior is already modeled directly?

## Collision Policy

What exact semantics and default threshold define an acceptably safe probabilistic collision bound, and how may advanced users override it?

## Distributed Deterministic Influences

How should the multiplicity or diversity of deterministic influences be specified or inferred across distributed scopes without burdening ordinary programs?

## Partial-Identifier Contract

What is the canonical serialization contract for partial probabilistic identifiers?

The contract must define when comparison can conclude definitely equal rather than possibly equal, and how identifiers produced under different optimization policies interoperate.

## Composite Identifiers

How are mixed structural and probabilistic identifier components represented in the type system, and how do their scopes and partitions compose?

## Injectivity Proofs

How should user libraries declare and prove full, argument-relative, and information-relative injectivity, and which primitive operations should carry compiler-known facts?
