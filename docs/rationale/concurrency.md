# Concurrency Decisions

This document records the reasoning behind the working [concurrency](../design/concurrency.md#concurrency-and-synchronization) model without presenting its unresolved planner as settled semantics.

## Synchronization As Capability

Acts already describe affected affine resources, so synchronization should build on that information rather than introduce an unrelated locking subsystem. Distinguishing mutation, syncable mutation, and satisfied synchronization lets types state what an operation can coordinate without prescribing one runtime primitive.

## Planning At The Execution Boundary

Choosing an acquisition plan near `synchronously run` may give the planner a more complete resource context than a reusable act definition has. Keeping permanent order out of the act type could improve composition, provided every nested syncable implementation can accept and propagate the chosen legal order. That proviso is a major unresolved proof obligation.

## Author Intent And Synchronized Reads

The compiler cannot infer that several observations are intended as one atomic snapshot. Requiring the author to express semantic grouping avoids manufacturing guarantees that were never requested.

Actual lock-taking reads are different: their synchronization affects deadlock even when no atomic snapshot was intended. Their mechanisms must therefore be visible to planning without classifying every read as synchronized.

## `syncing` And `locking`

`syncing` communicates surrounding context to an existing mechanism; it cannot create a mechanism for unsynchronized mutation. `locking` supplies lock-based capability where needed but should delegate when a resource already synchronizes itself, including through lock-free techniques.

Separating those roles permits nested composition to reuse an enclosing lock and elide an inner acquisition. This avoids self-deadlock without requiring every mutex to be reentrant.

## Ownership-Derived Ordering

Outer-before-inner acquisition follows meaningful ownership and lifetime structure, unlike arbitrary stable resource identifiers that would add nominal identity solely for lock ordering. The exact relation is not yet formal enough to guarantee that all compositions are comparable or stable under donation, so runtime tokens remain neither required nor entirely ruled out as a fallback.

## Derived Coordination

Explicit co-synchronization types are preferably avoided because the planner may derive regions and ordering from act composition. A dedicated type could multiply distinctions between using the same resources, using the same order, and holding locks over the same region. Avoiding it remains conditional on proving the planner complete enough for safe composition.

## Limited Manual Locking

Keeping raw locks away from ordinary code centralizes acquisition, ordering, and elision in transformations the compiler can analyze. Universal manual locking or universal reentrant mutexes would make deadlock prevention depend on programmer discipline and runtime behavior rather than checked plans.

## Diagnostics Rather Than Blanket Rejection

I/O while synchronized can cause long critical sections, but some valid atomic operations require it. A warning can expose the performance and scheduling risk without rejecting semantically legitimate programs.
