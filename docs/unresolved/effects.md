# Effect Questions

This document tracks questions left unresolved by the [effects](../design/effects.md#effects-resources-and-trusted-interfaces) design. Statements here are not guarantees until their answers are incorporated into the design.

## Act Contracts

How should act types describe dependencies, interference, mutation, isolation, duplication, elimination, reordering, and parallel execution precisely enough for static checking and optimization?

## Affine Declaration And Donation Syntax

What exact type and operator declarations represent affine ownership, containment, lifetime, donation, mutation, and consumption?

The single-owning-path and transfer semantics are settled, but representative `donate` syntax and operation-specific transfer forms are not final.

## View Creation

What expression creates `a view of X`?

Open possibilities include an explicit generic operator, implicit conversion when a parameter requires a view, or capability-specific forms. Nested views and capability weakening also need representation. `view X` is not settled syntax.

## Formal Lifetime Relation

What exact static relation represents owner/container and owned/contained lifetimes, including resources in aggregates, branches, and higher-order operations?

The relation must reject ownership cycles and invalid escape. **Ancestry** should not become formal terminology unless this model justifies it. Its synchronization consequences are tracked separately in [concurrency](concurrency.md#ownership-and-lock-order).

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
