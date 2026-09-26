# Concurrency Questions

This document tracks unresolved proof obligations and syntax in the working [concurrency](../design/concurrency.md#concurrency-and-synchronization) model. Nothing here is settled language semantics.

## Declarations And Operations

What exact structural declarations represent mutating, syncably mutating, syncly mutating, syncable, and synced acts?

What are the final signatures and grammar for `synchronously run`, `syncing`, and `locking`?

## Ownership And Lock Order

What static ownership or lifetime relation provides a consistent acquisition order?

Candidate ingredients include owner before contained resource, longer before shorter lifetime, and declaration or capture order. It remains unknown whether the relation is a tree, DAG, region order, total order along each execution path, or another structure. Values introduced in branches, collections, higher-order calls, and separate constructors must be addressed.

This relation is the major prerequisite for deadlock prevention without arbitrary stable resource identifiers.

## Donation And Reordering

Can donation or reparenting change the required order for a resource across program points?

If it can, the design must establish whether concurrent paths can observe incompatible valid orders, whether donation is a synchronization boundary, or whether ordering requires a more stable relation. Affine lifetime rules may rule out the conflict, but that has not been proven.

## Planning At `synchronously run`

Can every syncable operation remain sufficiently order-agnostic for an enclosing `synchronously run` to choose a legal order and propagate it through `syncing` and `locking`?

An internal operation may effectively force `A` before `B` while a surrounding composition requires `B` before `A`. The design needs either a proof that synchronization can always be safely lifted or reordered, or an additional capability or restriction describing when it can be.

## Eliminating Co-Synchronization Types

Can explicit co-sync declarations be omitted completely?

The preferred design assumes that lockable behavior is constructed through `locking`, `syncing` supplies outer context, enclosing locks can be lifted, supplied inner acquisitions are elided, and `synchronously run` sees all required resources. These assumptions require a compositional proof.

## Planner Rewrites

Which rewrites soundly compose sequential acts that mutate overlapping resources or combine lock-based and self-synchronizing operations?

Exploratory lowering examples are not final rules. In particular, the planner must prove when it can retain an outer lock around a nested operation and when an operation can accept synchronization context it did not choose itself.

## Lock Scope

When sequential operations use overlapping resources, should a safe plan retain a lock across both operations or release and reacquire it?

Holding it reduces lock operations but may reduce concurrency or worsen scheduling. Once safety is established, this should preferably remain an optimization choice rather than source semantics.

## Synchronized Reads

What effect representation makes lock-taking reads visible to deadlock analysis without pretending to infer atomic-snapshot intent?

Open questions include:

- whether **syncably reading** and **syncly reading** are the right terms;
- whether every read of shared mutable affine state is an act;
- how lock-free acquire or atomic reads are represented;
- how read and write requirements combine;
- what thread-safety guarantees apply to observations.

## Lock-Free Semantics

What atomic, fence, and memory-order guarantees allow lock-free syncable resources to compose with lock-based resources without unnecessary locking?

`syncing` is expected to delegate to lock-free mechanisms while `locking` handles mutex-like resources, but the memory model has not been designed.

## Stable Resource Identities

Can the final ordering model avoid stable runtime resource identifiers entirely?

Such tokens are not accepted architecture because they fit poorly with structural typing. They remain only a possible fallback if no lifetime-derived relation can support safe composition.

## Priority Inversion And I/O

What diagnostics or policies should address priority inversion and external I/O while synchronization is held?

Priority inversion is deferred. I/O may warrant warnings for long critical sections, but blanket rejection is too strong.
