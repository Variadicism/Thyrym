# Effect Decisions

This document records the reasoning behind Thyrym's [effects](../design/effects.md#effects-resources-and-trusted-interfaces) without extending their guarantees.

## Precise Acts

Acts expose effects and dependencies through types instead of hiding all interaction behind an opaque IO category. The compiler can preserve required observable behavior while potentially reordering, parallelizing, eliminating, or duplicating work when contracts prove those transformations safe.

Errors remain explicit because hiding exceptions inside acts would undermine Thyrym's requirement that runtime failure possibilities be represented and handled as values. Effect wrappers such as isolation or synchronization change how effects are observed, not which result cases exist.

## Exceptional Unsafe Effect Escape

Unsafe effect escape is deliberately exceptional because ordinary observable interaction should remain explicit in act types. Its purpose is to let a developer assert a property the compiler cannot prove, not to provide a convenient default route around effects or failure handling. Diagnostic instrumentation is a motivating use, but printing can still block or fail, so any diagnostic operation needs a narrow and truthful failure policy.

The three proposed clauses describe programmer-facing intentions rather than inheriting implementation behavior from similarly motivated operations in another language. Shared lazy evaluation and a concurrency nonduplication guard may be one way to support `unsafely once`, so it is not assumed to require an external memo table or a wholly independent primitive from `unsafely each time`. A stronger exactly-once language guarantee could still require optimizer restrictions or compiler cooperation.

Likewise, `unsafely each time` means that duplication is permitted, not that execution necessarily occurs on every use. `unsafely recursively` describes a lazy effectful fixpoint; it neither subsumes hidden deferred I/O in general nor establishes that streaming needs such a fixpoint.

## Explicit Streaming Instead Of Hidden Deferred I/O

Hidden on-demand I/O can move a read failure beyond the act and resource scope that opened the file. It also obscures effect order and makes it difficult to state when the handle closes after cancellation or early termination. An explicit demand-driven stream keeps every request effectful while still permitting bounded-memory incremental processing.

Ordinary recursion and explicitly sequenced producer and consumer steps are sufficient for the basic architecture. Performance should be measured before adding an unsafe primitive. Lazy heavyweight initialization can instead use an explicitly requested effectful once-cell, while sensors, entropy, foreign calls, logging, and profiling should retain timing and effects in their contracts.

## Other Unsafe Facilities

No current use case requires a separate local-state monad or unsafe state-to-I/O conversion, a general raw type coercion, or specialized unsafe mutable-array primitives. This is provisional rather than a claim that such facilities can never be needed. Local mutation still requires a sound non-escape story, safe representation-preserving conversion is distinct from arbitrary coercion, and foreign mutation must be governed by contracts precise enough for the compiler to copy or prove destructive reuse unobservable.

## Single Affine Ownership

One owning path prevents accidental aliasing and conflicting mutation while permitting deliberate ownership transfer. Donation makes long-lived escape explicit rather than allowing a captured value to create a hidden second owner. Views separate non-owning access from transfer.

Containment constrains lifetime because an owned resource cannot outlive the owner that makes its use valid. Child-to-parent cycles would make that ordering incoherent.

## Capability-Relative Affineness

Affineness follows the capabilities that require an affine resource instead of every physical field in a representation. This preserves structural typing: a value can require affine use for pool-backed list operations while still exposing an unrelated pure capability that does not touch the pool.

Pure descriptions remain pure when acts use them to produce resources. Otherwise, immutable values such as paths would acquire ownership restrictions merely because an effectful operation can consume them as input.

## Mutation Vocabulary

Mutation and consumption are separate because transformation-and-return preserves an owning path, while consumption ends it. Calling every affine argument consumed would erase information needed to reason about subsequent ownership and lifetime.

Invariant mutation is defined relative to visible type guarantees rather than byte-for-byte restoration. Hidden representation is not semantically important when every promised capability and refinement remains satisfied. Call-site types may expose different guarantees, so the same operation can be invariant for one use and non-invariant for another.

## Isolation

**Isolated** replaced **ephemeral** because the property concerns semantic relevance and confinement rather than duration. Isolation can depend on application policy: the compiler can enforce a declared policy but cannot infer that logging or another real-world effect is unimportant to its author.

A separate persistent-act type would add no capability. An ordinary act already represents the absence of an isolation guarantee.

## Steadiness

**Steady** replaced **quasi-pure** because the act remains effectful and executing; only the guarantees visible through the relevant type remain stable. The term avoids claiming literal purity while explaining why pure-looking composition can be sound.

Steadiness cannot license arbitrary deletion. Required effects remain program obligations even when their mutations are isolated, synchronized, and invariant to the observed type.

`reveal` is limited to exposing hidden steady effect structure because isolation changes whether a context observes an effect. Synchronization instead satisfies or transforms a concurrency requirement and is not hidden in the same sense.

## Capture Injection

Capture injection redirects an affine resource rather than source organization. Requiring a surrogate to satisfy every original capability preserves structural soundness while allowing its physical backing to differ. Dispatch through the captured surrogate keeps redirection stable even when calls pass through several library layers.

It complements rather than replaces import injection: import injection is general but structurally coupled to modules, while capture injection is narrow but robust for resource-level substitution.

## Blocking Is Not An Effect

Time spent waiting is not observable interaction by itself; expensive pure computation can delay a caller in the same way. Classifying all blocking as effectful would conflate progress with effects. Waiting becomes relevant when its resource and synchronization interactions can affect starvation or deadlock.

## Foreign Code As A Trusted Boundary

Thyrym cannot prove arbitrary foreign implementations safe or contract-compliant. Treating foreign declarations as explicit trusted boundaries preserves useful interoperability without weakening the meaning of guarantees made by safe Thyrym code.
