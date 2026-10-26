# Resource Lifetime, Ownership, And Effect-Scope Questions

This document tracks resource-lifetime questions left unresolved by the [effects](../design/effects.md#effects-resources-and-trusted-interfaces) design. Statements here are not guarantees until their answers are incorporated into the design.

The established single-owning-path model, explicit donation, and non-owning views remain the starting point. The open problem is how those semantics apply transitively through mutable storage, closures, callbacks, independent execution, and composed effect scopes.

## WebSocket Escape Acceptance Case

The primary acceptance case is an application whose long-lived mutable keyboard state and active WebSocket jointly support multi-key shortcuts and updates to a web control panel.

Suppose a `with` scope opens the WebSocket and code attempts to place it in a mutable cell that outlives the scope. Merely ending the lexical scope must not leave an apparently usable handle after its underlying resource has been closed. The design must either reject that storage, establish that the destination cannot outlive the resource, or perform an explicit donation that transfers both ownership and the relevant cleanup obligation to a valid longer-lived owner.

The same rules must still permit ordinary helper calls, state mutation, callbacks, and effect composition needed by the application.

## Scope And Transfer

- What exactly does a `with` scope own: the underlying resource, a capability, the handle value, the cleanup obligation, or some combination?
- When ownership is donated out of a scope, how does the scope relinquish closure responsibility, and what statically ensures that the recipient eventually consumes the resource?
- How are temporary views passed to helper functions, and what prevents a view from escaping through a return value, mutable cell, closure, callback, or independently executing task?
- How are lifetime requirements represented transitively in arbitrary aggregates and higher-order values rather than checked only by lexical name lookup?

## Mutable Storage And Concurrency

- How does a mutable cell declare the lifetime bounds of resources or views it may store?
- Can a closure capture an owning resource or a view, and how is the closure's own lifetime constrained?
- What rules apply when a callback or asynchronous chamber may continue after the creating scope exits?
- What are the usability and performance costs of lifetime checks and explicit donation in common stateful applications?

## Composable Effects And Cleanup

Narrow resource-specific effect contexts are insufficient when one operation needs networking, random numbers, clocks, and mutable state together. Composable effect or capability information may describe what operations are available, but capability presence alone does not prevent a handle from being stored beyond its lifetime.

- How do nested effects compose while preserving ownership and lifetime checks?
- How are clean shutdown, cancellation, I/O failure, and finalizer failure represented under explicit-failure guarantees?
- Which guarantees belong to the resource type, its owner, an effect capability, or the surrounding act?
