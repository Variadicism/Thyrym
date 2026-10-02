# Thyrym Core Foundations

> **Status:** Early foundational documentation  
> This document describes Thyrym's central goals and mental model. Detailed systems are documented separately from this core overview.

The reasons behind foundational choices are recorded in [core](../rationale/core.md#core-decisions). Foundational questions that remain open are tracked in [core](../unresolved/core.md#core-questions).

## Overview

**Thyrym** is a general-purpose, pure functional programming language built around several closely related goals:

- express programs in a readable, English-inspired syntax;
- allow users to define new syntax through operators;
- use a structural type system to represent both what values are and what is known about them;
- eliminate language-level runtime errors by proving correctness requirements during compilation;
- represent side effects explicitly through monads and typed acts rather than arbitrary mutation;
- preserve enough semantic information for aggressive evaluation, specialization, reordering, and optimization;
- keep the irreducible language core as small as practical.

Thyrym currently targets C, although the design may eventually support other compilation targets.

Thyrym is built on a shared language engine called **Thyry**.

## Thyry, Thyrym, And Aryya

Thyrym and Aryya are separate languages built on the same underlying engine.

### Thyry

**Thyry** provides shared infrastructure including parsing, pattern matching, type resolution, mostly structural type matching, invariants, operator resolution, and other compiler and interpreter machinery.

Thyry is semantic and compiler infrastructure rather than a user-facing language.

### Thyrym

Thyrym is a general-purpose pure functional language. Its intended primary compilation target is C, and it permits ordinary runtime-sized data and general-purpose computation.

### Aryya

Aryya is a specialized language intended primarily for vectorized mathematical computation. It shares substantial type-system and parsing machinery with Thyrym through Thyry but has different syntax, semantics, constraints, and compilation targets.

Concepts must therefore be classified carefully: some belong to Thyry and naturally appear in both languages, while others belong specifically to Thyrym or Aryya. Similarity does not imply identical language semantics.

## No Language-Level Runtime Errors

One of Thyrym's primary goals is:

> **Valid Thyrym code should not encounter a language-level runtime error.**

This does not mean the outside world cannot fail. A requested file may not exist, a service may be unavailable, or an API may reject a request. These are legitimate runtime conditions outside Thyrym's control.

Every such possibility must be represented explicitly and accounted for by valid code. Failures are values rather than exceptions.

Failures caused by invalid program state should be rejected during compilation. Examples include:

- dereferencing an invalid value;
- accessing an array outside proven bounds;
- violating an operator precondition;
- using a value without satisfying its invariants;
- failing to account for an explicitly possible error;
- throwing an uncaught exception.

Thyrym has no ordinary exception-throwing mechanism. Error composition is described in [monads](monads.md#monads-and-contextual-binding).

### Trusted Boundaries

Some guarantees necessarily depend on trusted information. Important examples include foreign-function contracts and explicitly assumed invariants.

Incorrect trusted information can undermine Thyrym's safety guarantees. Foreign code therefore crosses an unsafe, trusted boundary analogous in spirit to Rust's `unsafe`; Thyrym does not attempt to prove arbitrary C code memory-safe. Trusted interfaces are described in [effects](effects.md#foreign-functions), and invariant assumptions are described in [types](types.md#trusted-assumptions).

## Pure Functional Semantics

Ordinary computation is referentially transparent: replacing a pure expression with its value must not change program meaning.

Mutation and observable interaction with the outside world are represented explicitly through typed monadic operations, particularly acts. Their contracts and resource semantics are described in [effects](effects.md#effects-resources-and-trusted-interfaces).

### Unspecified Evaluation Strategy

Thyrym does not define whether ordinary computation is evaluated strictly, lazily, during compilation, in parallel, or through a mixture of strategies.

A valid implementation may choose any evaluation strategy that preserves program semantics. This includes freedom to memoize or recompute pure work, fuse or inline operations, and perform semantics-preserving rewrites. The language specifies results, dependencies, and guarantees rather than one schedule for producing them. Observable behavior must continue to satisfy declared types, invariants, effects, affine-resource contracts, convergence and stoppability guarantees, and every other explicit semantic requirement.

A strict evaluation model may be a useful intuition, but it is not a language guarantee. Nor does Thyrym require bit-for-bit identity between all valid implementations when a contract permits multiple results, such as compile-time and platform floating-point approximations that both meet the declared numeric requirements.

Tail-call lowering is likewise an implementation and performance concern rather than a source-level semantic guarantee. The compiler should eventually perform useful recursion-to-loop or accumulator transformations itself instead of relying entirely on its C target.

## The Irreducible Core

Thyrym is intended to have a very small built-in core. The exact list is deliberately not finalized; implementation should reveal which pieces truly must be primitive.

Current likely core elements include:

- the `bits` type;
- operator declaration and definition machinery;
- Thyry's underlying pattern facilities;
- `built using`;
- `run with`;
- enough type machinery to bootstrap user-defined types;
- facilities necessary to report compilation failure.

Many constructs that might be compiler primitives elsewhere are intended to be ordinary operators. Possible examples include conditionals, loops, variable bindings, monadic binding, contextual DSL constructs, and imports.

The exact bootstrap inventory remains provisional until implementation demonstrates what must be intrinsic. The relevant questions are tracked in [core](../unresolved/core.md#irreducible-core).

## Design Map

The major systems built on this foundation are:

- extensible syntax, contextual grammar, and callable behavior in [operators](operators.md#operators-and-grammar);
- structural types, properties and evidence, polymorphic relationships, invariants, primitive data, and compile-time knowledge in [types](types.md#types-and-compile-time-knowledge);
- effect and resource contracts in [effects](effects.md#effects-resources-and-trusted-interfaces);
- the working synchronization and deadlock-prevention model in [concurrency](concurrency.md#concurrency-and-synchronization);
- source organization and imports in [modules](modules.md#modules-and-imports);
- monadic composition and contextual binding in [monads](monads.md#monads-and-contextual-binding);
- convergence, pursuits, stopping, and asynchronous containment in [pursuits](pursuits.md#pursuits-convergence-and-stopping);
- scoped uniqueness guarantees in [uniqueness](uniqueness.md#scoped-uniqueness).

## Core Mental Model

A useful way to think about Thyrym is:

> **Thyrym is a tiny operator-based functional language in which grammar, types, and contextual knowledge are all extensible through the same underlying machinery.**

Operators do more than compute values. They can define grammar, accept operators and unresolved pre-ops, introduce operators into a continuation, refine visible types, generate contextual DSLs, establish generic relationships, represent pure transformations, and describe explicit effects.

Types are not nominal labels attached to values. They are first-class values describing structural capabilities and invariants. A value satisfies a type when the compiler can prove that it provides everything the type promises.

The language preserves those semantics for as long as possible so correctness checking and optimization follow from the same rich information.

Thyrym's central aspiration is:

> **Invalid program states can be rejected at compilation, while valid programs leave the implementation broad freedom in how they are executed.**
