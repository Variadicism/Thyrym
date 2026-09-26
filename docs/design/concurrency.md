# Concurrency And Synchronization

> **Status:** Working design, not settled language semantics  
> Affine ownership and effect visibility are established prerequisites, but synchronization ordering and composition still require proof.

Thyrym aims to derive synchronization from affine ownership, effect descriptions, and structural capabilities while keeping manual lock management away from ordinary source. The current vocabulary and constraints are documented here so later work can refine them without presenting the lock planner as complete.

The rationale is recorded in [concurrency](../rationale/concurrency.md#concurrency-decisions). Open proof obligations and syntax are tracked in [concurrency](../unresolved/concurrency.md#concurrency-questions).

## Synchronization Vocabulary

**Syncable** describes an operation capable of satisfying synchronization requirements. Mutation information may progress through these conceptual forms:

- **mutating**: mutation occurs without a claim that it can be made safe for concurrent execution;
- **syncably mutating**: the operation provides a synchronization mechanism a caller can coordinate;
- **syncly mutating**: the relevant synchronization requirements have been satisfied.

A conceptual structural model is:

```text
an act mutating
    Mutables, some affine values

an act syncably mutating
    Syncables, some affine values
    Mutables, some affine values

a syncable act
    where Mutables = Syncables

an act syncly mutating
    Synced, some affine values
    Syncables, some affine values
    Mutables, some affine values

a synced act
    where Synced = Syncables
```

These are conceptual capability relationships rather than final declarations. A syncable act can become synced by satisfying all its synchronization requirements. Possible errors remain explicit in its result type throughout.

## `synchronously run`

The working operation:

```thyrym
synchronously run Act
```

performs the formal transition from a syncable act to a synced act. It is expected near asynchronous execution boundaries before an act may run concurrently.

Nested requirements compose. If later composition introduces a syncable mutation whose requirements are not yet satisfied, the combined act is no longer fully synced and requires another synchronization transition.

A promising but unresolved direction is for `synchronously run` to choose a resource order using the complete synchronization context and propagate that plan through `syncing` and `locking`. Act types would then describe what can and must synchronize without permanently encoding one acquisition order.

## Reads And Synchronization

The compiler cannot infer which independent observations an author intends to form one atomic snapshot. Semantic grouping of reads remains an explicit author-level choice.

A read that acquires a lock nevertheless participates in acquisition ordering and can contribute to deadlock. The effect model therefore needs to expose actual synchronization performed by reads, potentially through concepts such as **syncably reading** and **syncly reading**, without placing every ordinary read in one global synchronization set. The exact representation is unresolved.

## `syncing`

`syncing` tells a synchronization-capable act which affine values participate in its surrounding synchronization context. It is intended to:

- coordinate nested synchronization mechanisms;
- let lock-free implementations ignore irrelevant lock context;
- let lock-based implementations respect an enclosing acquisition plan;
- avoid inventing synchronization for an unsyncable mutation.

Conceptually, asking an act already syncable for `A` to sync `A` delegates to its mechanism. Supplying additional context is valid only when the act can coordinate every required resource in a safe order. An act that merely mutates `A` without synchronization capability cannot become safe through `syncing A`; it requires `locking` or another mechanism.

Exact signatures and ordering checks are not settled.

## `locking`

`locking` makes otherwise unsynchronized mutation syncable by supplying lock-based coordination, while delegating to an existing synchronization mechanism when one is available. It does not necessarily mean immediately acquiring every listed lock.

The transformation should:

- avoid locking a resource that safely synchronizes itself without locks;
- avoid acquiring the same non-reentrant lock twice;
- permit an outer plan to provide a lock an inner act would otherwise acquire;
- acquire owning or outer resources before dependent inner resources when that ordering is established;
- specialize composition so locks are held no more broadly than the selected safe plan requires.

For a plain mutation, the conceptual result may lock, run the act, and unlock. For an already syncable operation, `locking` should delegate through `syncing` rather than unnecessarily place it behind a mutex. If an enclosing plan already holds an inner lock, the inner acquisition is elided instead of requiring universal reentrant mutexes.

Locking an unmutated outer resource can be legitimate solely to acquire an inner resource in a safe order.

### Illustrative Planning Shapes

The following shapes record the current planning direction; they are examples to prove or refine, not normative lowerings:

```text
locking A, mutable A
    => lock A; do Act; unlock A

locking A, syncable A
    => syncing A, do Act

locking A and B, mutable B
    => lock A; lock B; do Act; unlock B; unlock A
```

If an enclosing `locking A and B` supplies both locks, nested lockable behavior for `B` should not reacquire `B`. A lock-free `B` should not force acquisition of an otherwise unneeded `A` mutex merely because `A` precedes `B` in a lock order.

For sequential operations, disjoint requirements may be handled separately. A mutation of `A` followed by self-synchronizing work on `B` need not retain `A` solely because it appeared earlier. When later work mutates both `A` and `B`, a planner may hold `A` across both steps and acquire `B` only for the second, or may release and reacquire `A`; choosing lock scope remains unresolved.

The composition rewrites and their proof are unresolved. Examples must not be treated as final lowering rules.

## Ordering Direction

The current safety direction is to acquire an owning or outer affine resource before an owned or contained inner resource. A lifetime- or ownership-derived order is preferred to an arbitrary runtime resource identifier.

The exact relation may involve ownership, containment, lifetime, declaration, or capture order. **Ancestor** is not finalized terminology, and the design does not currently require stable unique lock-order tokens.

Whether an enclosing `synchronously run` can always select and propagate a compatible order remains the central open proof obligation. Ownership donation may also alter containment relationships and must be reconciled with any derived order.

## Composition Goals

The preferred design avoids a separate structural property meaning that resources are permanently co-synchronized. Instead, act types describe mutation, reading, and synchronization capability while a planner derives coordinated regions from composition and ownership information.

This remains conditional on proving that:

- enclosing locks can be lifted where required;
- inner acquisitions supplied by the enclosing plan can be elided;
- independently implemented syncable operations can accept a compatible larger plan;
- lock-free and lock-based mechanisms compose under a sound memory model.

No manual-lock syntax, stable runtime resource identity, explicit co-sync type, or universal reentrant-lock requirement is settled by this working design.

## Diagnostics And Deferred Concerns

External I/O while synchronized may deserve a warning because it can lengthen critical sections, but valid atomic operations may require I/O, so it is not a blanket error.

Priority inversion and exact atomic, fence, and memory-order semantics are deferred.

`reveal` exposes hidden steady effect structure; it does not undo or bypass synchronization planning, as described in [effects](effects.md#reveal).
