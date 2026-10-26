# Effects, Resources, And Trusted Interfaces

Thyrym represents observable interaction explicitly through typed acts and uses affine ownership for resources and uniquely held stateful authority. The rationale is recorded in [effects](../rationale/effects.md#effect-decisions), and unsettled details are tracked in [effects](../unresolved/effects.md#effect-questions).

## Affine Ownership

Affine values represent resources with ownership and lifetime obligations. An affine value has one owning path: it cannot be freely duplicated, and authority over it cannot simultaneously exist in independent owning locations. Ownership may be transferred, while non-owning access may use a view.

### Donation

**Donation** transfers ownership of an affine value. The previous owner cannot continue using the value as though it retained ownership. Long-lived escape therefore occurs through an ownership transfer rather than a hidden second owner.

Representative syntax is:

```thyrym
donate WebSocket to Pool
```

The semantics are settled, but this spelling and operation-specific transfer forms are not final syntax.

### Views

A **view** permits non-owning access without transferring ownership. Its type-level form is:

```thyrym
a view of File
```

For example, an operation may accept:

```thyrym
\get file size 'File, a view of File'
    => an act making an integer
```

The expression syntax that creates a view is unresolved. In particular, `view File` is not established syntax.

### Lifetime And Containment

An affine value may own or contain another resource. The owner or container must remain valid for every resource whose ownership depends on it, and child-to-parent ownership cycles are invalid. The exact formal lifetime relation remains under design; **ancestry** is not established terminology for it.

A containing value behaves affinely when an exposed capability cannot be fulfilled without an uncontained affine resource. Mere physical storage of an affine field does not make every capability of the container affine.

For example, list operations on a thread-pooled list may depend on its affine pool and therefore require affine use. The same value may still satisfy a pure semigroup capability if concatenation does not use that pool. Affineness follows the capabilities that require ownership-sensitive state.

A pure value may produce an affine resource without becoming affine itself. A path remains a pure immutable description even when an act uses it to open an affine file, because the path's own capabilities do not depend on the live file.

## Mutation And Consumption

An affine **mutation** takes a resource and returns a transformed surviving form of that resource. **Consumption** takes a resource and returns no surviving form, ending its lifetime. Closing a file, killing an independently executing resource, and finally awaiting an exclusively controlled pursuit are examples of consumption. Ordinary transformation-and-return is mutation rather than consumption.

Mutation information belongs to the act's effect description so callers and the compiler can determine what state may change.

### Invariant Mutation

An **invariant mutation** preserves every assurance and capability visible through the relevant type. It need not restore hidden representation byte-for-byte.

This property is type-relative and may be established at a call site. Appending data may preserve the guarantees of `a file` while invalidating a refinement such as “a file ending with X.” A mutation counts as invariant for a use only when its post-mutation value still satisfies every requirement observed through that use's type.

The former term **briefly mutating** is superseded by **invariantly mutating**.

## Acts And Explicit Failure

An **act** is a monadic representation of operations that may produce observable effects. Opening, reading, writing, or otherwise interacting with external systems commonly involves acts.

The IO-like type phrase is:

```thyrym
an act making A
```

Acts do not contain hidden exceptions. Every possible failure remains explicit in the type of the value made by the act:

```thyrym
an act making an error or a user
```

Isolation, steadiness, synchronization, or another transformation cannot erase an error possibility.

Act contracts should expose effects more precisely than one opaque IO category so the compiler can determine interference. When contracts prove the transformation sound, an implementation may reorder or parallelize acts and eliminate or duplicate work whose required effects, resources, and results are preserved. Acts specify obligations and dependencies rather than one imperative schedule. Their composition is described in [monads](monads.md#monads-and-contextual-binding).

## Unsafe Effect Escape Clauses

> **Working design:** These names and purposes are preferred, but their exact semantics are not yet language guarantees.

**The Unholy Trinity** is the working name for three exceptional clauses that allow an effectful computation to cross an otherwise pure boundary:

| Clause | Intended purpose |
| --- | --- |
| `unsafely once, Act` | Present the result of a shared effectful computation as a pure value without duplicating that shared computation. |
| `unsafely each time, Act` | Present the result of an effectful computation as a pure value without a nonduplication guarantee. |
| `unsafely recursively, Builder` | Construct an effectful value whose builder can refer lazily to its own eventual result. |

The comma is part of each proposed clause form. The names do not by themselves define what identifies a shared computation, whether an act must execute, which transformations may duplicate it, or which evaluation guarantees a recursive reference receives. Those questions remain open.

These clauses are explicit developer assertions for exceptional cases where the compiler cannot establish a required property. They are not the ordinary interface for printing, file access, sockets, operating-system randomness, or foreign interaction. Unsafe use does not erase possible failure or automatically preserve totality; its contract must distinguish explicit failure results from properties trusted to the developer. The outstanding semantics are tracked in [effects](../unresolved/effects.md#unsafe-effect-escape-semantics).

No general hidden deferred-I/O clause is currently proposed. In particular, the current API does not include an operation that turns on-demand file reads into an apparently pure lazy string.

## Explicit Demand-Driven Streams

Incremental I/O uses an explicit effectful stream. A producer supplies a chunk in response to a request made within an effectful computation, while pure transformations may process chunks after they are received. Reads, errors, effect ordering, and resource cleanup therefore remain visible to the surrounding act protocol.

The following is an illustrative API sketch, not established Thyrym syntax:

```thyrym
with file "input.txt" as Source,
    for each Chunk received from Source,
        process Chunk
```

A stream protocol must represent read failure explicitly and define EOF, cancellation, early termination, handle closure, and cleanup failure. It may process data incrementally without representing secretly effectful contents as a pure `String`. High-performance implementations may use producer and consumer state machines, closures, continuations, and effectful steps; the model does not require hidden deferred I/O or recursive effect fixpoints.

Holding a file open does not by itself guarantee that another process cannot modify it. Stability or purity claims require an explicit platform or resource contract rather than an assumption derived from an open handle.

## Isolated Acts

An **isolated act** is an act whose relevant side effects are treated as confined or semantically irrelevant under the current policy. Isolation may express author intent or configuration rather than an intrinsic fact discoverable by the compiler.

Thread bookkeeping may commonly be isolated, while arbitrary file writes ordinarily are not. An application may deliberately isolate selected logging, and testing or release configuration may suppress it across a call stack. The compiler enforces the consequences of a declared policy but cannot prove the human claim that an effect does not matter.

Isolation does not erase errors. Omitting or redirecting an effect can change later success, so every failure outcome remains typed.

There is no separate **persistent act** category. An ordinary act simply lacks the isolation property. The former term **ephemeral act** is superseded by **isolated act**.

## Steady Acts

A **steady act** is conceptually an isolated act whose relevant mutations are synchronized as required and invariant relative to the type context through which it is observed. It may participate in pure-looking composition because all semantically visible guarantees remain stable, but the act still executes and its effects still occur.

Like invariant mutation, steadiness is type-relative. A hidden change can be steady for a caller that observes only preserved capabilities while failing to be steady for a caller that depends on a changed refinement.

**Steadily** and **steadiness** are the related forms. **Quasi-pure** is superseded terminology.

### `reveal`

> **Working design:** The exact operator and type formalization remain open.

`reveal` exposes the act and effect structure hidden by steadiness and can override an isolation assumption where a caller needs to inspect the underlying effects. Lower-level code may treat an effect as isolated while a higher-level caller reveals it.

Conceptually, a pure-looking operation may reveal as:

```thyrym
a steady act mutating Pool of List making a list of Bs
```

`reveal` is specific to hidden steady or isolated effect structure. It is not a general operation for undoing synchronization transformations.

## Capture Injection

> **Working design:** The capability requirement is established, but declaration and application syntax remain open.

**Capture injection** replaces a captured affine resource with a surrogate satisfying the same capabilities while redirecting its physical behavior. A test may preserve a file's logical path as `test.txt` while its surrogate maps operating-system operations to `temp/test.txt`.

The injected resource must satisfy every capability and law promised by the original affine type. Subsequent operations dispatch through the surrogate, keeping the redirection stable throughout the call stack. Capture injection specifically addresses affine capture; compile-time redirection of arbitrary dependencies is the separate import-injection design described in [modules](modules.md#import-injection).

## Blocking

Waiting or blocking is not itself a semantic side effect merely because obtaining a result takes time. Pure expensive computation also delays its caller. Blocking becomes relevant to concurrency when it interacts with synchronization, resource starvation, or deadlock.

External I/O under synchronization may warrant diagnostics for long critical sections, but it is not categorically invalid because legitimate atomic work may require I/O. The working synchronization model is described in [concurrency](concurrency.md#concurrency-and-synchronization).

## Independent Execution

Some resource contracts describe computation that progresses independently of the current path. Pursuit design provisionally calls such a compiler-significant resource an **asynchronous chamber**. It is affine, cannot be manufactured by ordinary pure code, and may be created by a trusted foreign or system operation. Threads, processes, and tasks may provide concrete forms.

An operation that consumes a chamber and establishes that its execution has ended is a kill-like operation. Consumption alone does not imply termination; the operation's contract must explicitly guarantee that the independent execution cannot continue or resume. Forced termination must also expose truthful post-state guarantees for every affected resource. Detailed containment and stopping semantics are described in [pursuits](pursuits.md#asynchronous-containment).

## Program Completion

A terminating program is complete only after its required top-level acts and outstanding effect obligations have completed. A pure unused computation, including an unused pursuit, may be discarded; an effectful obligation may not be discarded merely because no result uses its value. Value availability therefore does not by itself imply that all independent effects have finished.

## Foreign Functions

Thyrym expects to use foreign functions, especially C functions, for low-level and standard-library functionality.

Foreign declarations must provide detailed contracts describing behavior relevant to Thyrym's correctness model, potentially including:

- accepted inputs;
- possible outputs and explicit failures;
- side effects and resource mutations;
- ownership, donation, views, and lifetime implications;
- isolation, synchronization, and independent-execution behavior.

### Trusted Boundary

Thyrym cannot prove that arbitrary foreign code is memory-safe or obeys its declared contract. A foreign declaration therefore establishes an unsafe, trusted boundary.

Incorrect contracts or implementations can invalidate guarantees made by otherwise-safe Thyrym code. The language should not prohibit useful C techniques merely to make arbitrary foreign code statically provable.

Future compilation targets may require trusted interface mechanisms analogous to the C foreign-function boundary.
