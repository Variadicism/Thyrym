# Effect Questions

This document tracks questions left unresolved by the [effects](../design/effects.md#effects-resources-and-trusted-interfaces) design. Statements here are not guarantees until their answers are incorporated into the design.

## Act Contracts

How should act types describe dependencies, interference, mutation, isolation, duplication, elimination, reordering, and parallel execution precisely enough for static checking and optimization?

## Unsafe Effect Escape Semantics

What exact guarantees do the proposed `unsafely once,`, `unsafely each time,`, and `unsafely recursively,` clauses provide?

For `unsafely once`, what identifies one shared computation: a lexical expression, an instantiated call, a runtime suspension, or an explicitly named operation? Does the clause promise only that concurrent evaluation of one shared suspension is not duplicated, or programmer-visible exactly-once execution? Which compiler transformations may omit, duplicate, merge, float, or reorder it?

For `unsafely each time`, when may evaluation be duplicated, and can an undemanded result avoid execution? A true per-invocation operation would be a stronger and different contract.

For both clauses, how do parallel evaluation and pure-value sharing interact with execution? Are explicit error results memoized when the surrounding computation is shared? Which unchecked failures, if any, can cross the unsafe boundary, and what narrowly defined policy applies to best-effort diagnostic effects?

For `unsafely recursively`, what language-level laziness guarantee prevents the self-reference from being forced before publication? What are the failure and concurrency semantics if premature forcing nevertheless occurs? Can the operation be implemented from ordinary lazy closures, an unsafe effect-to-value conversion, and a correctly managed single-assignment placeholder, or does it need another primitive? A compiler optimization hint is not sufficient for correctness-critical laziness, and no need for this operation has yet been established by streaming.

The three clauses are not assumed to exhaust every future unsafe facility. Any additional primitive needs a concrete requirement from areas such as foreign interaction, concurrency, memory management, representation, or optimizer contracts.

## Effectful Stream Protocol

What structural types and operations describe explicit demand-driven producers and consumers? The protocol must define EOF, cancellation, early termination, closure, and how read and cleanup errors compose without turning failures into hidden exceptions.

Can an effectful once-cell provide lazy heavyweight initialization while preserving explicit request timing and failure handling? Its ownership, retry, cancellation, concurrency, and failure-memoization behavior remain to be designed.

## Affine Declaration And Donation Syntax

What exact type and operator declarations represent affine ownership, containment, lifetime, donation, mutation, and consumption?

The single-owning-path and transfer semantics are settled, but representative `donate` syntax and operation-specific transfer forms are not final.

## View Creation

What expression creates `a view of X`?

Open possibilities include an explicit generic operator, implicit conversion when a parameter requires a view, or capability-specific forms. Nested views and capability weakening also need representation. `view X` is not settled syntax.

## Formal Lifetime Relation

What exact static relation represents owner/container and owned/contained lifetimes, including resources in aggregates, branches, and higher-order operations?

The relation must reject ownership cycles and invalid escape. **Ancestry** should not become formal terminology unless this model justifies it. Detailed acceptance cases for mutable storage, callbacks, threads, and composed effects are tracked in [resource lifetimes](resource%20lifetimes.md#resource-lifetime-ownership-and-effect-scope-questions). Its synchronization consequences are tracked separately in [concurrency](concurrency.md#ownership-and-lock-order).

## Isolation Policy

How are isolated effects selected, scoped, overridden, and configured across a call stack?

The compiler can enforce a declared policy but cannot infer the author's claim that an effect is semantically irrelevant.

## Steady Act Formalization

How do act types express isolation, required synchronization, and call-site-relative invariant mutation strongly enough to establish steadiness?

The concept is settled, but its exact structural declaration and inference rules may evolve.

## `reveal`

What exact operator and type transformation exposes an act hidden by steadiness, and how does it override isolation policy?

It must not become a general operation for undoing synchronization.

## Capture Injection

What syntax and scope select a captured affine resource for replacement, and how is the surrogate's satisfaction of every promised capability checked?

The injected resource may redirect physical behavior while retaining logical properties, but the application mechanism remains a working design.

## Foreign Contracts

What exact syntax and semantics describe foreign inputs, outputs, explicit failures, effects, mutation, ownership, donation, views, lifetimes, isolation, synchronization, and independent execution?

Foreign declarations must remain explicit trusted boundaries because Thyrym cannot prove arbitrary implementations obey their contracts.

How do foreign array and pointer contracts describe aliasing, mutability, ownership, concurrency, and read-only promises precisely enough to decide whether copying is required or destructive reuse is unobservable?
