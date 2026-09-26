# Pursuits, Convergence, And Stopping

Thyrym represents potentially ongoing computation with **pursuits**. This chapter defines demand-sensitive convergence, cooperative stopping, awaiting, pursuit composition, and containment in independently executing resources. Pursuits use the existing monadic and affine-resource models rather than introducing separate binding or ownership rules.

The reasons behind these choices are recorded in [pursuits](../rationale/pursuits.md#pursuit-decisions). Deferred details are tracked in [pursuits](../unresolved/pursuits.md#pursuit-questions).

## Convergence And Divergence

An ordinary value should not hide the possibility that the demanded computation never makes that value available. Thyrym therefore treats convergence and divergence as explicit semantic concerns while recognizing that general termination is undecidable.

Analysis has at least three outcomes:

1. **proven convergent**;
2. **proven divergent**;
3. **unknown**.

Failure to prove convergence does not prove divergence, and failure to prove divergence does not prove convergence. The compiler's proof repertoire is intentionally extensible.

The existing assumption mechanism applies unchanged. A proven convergence claim needs no assumption, a disproven claim fails compilation, and an unknown claim may cross an explicit `assuming ...` trust boundary. An assumption cannot override a contradiction.

### Demand-Sensitive Convergence

Convergence is relative to the demanded result. An infinite or corecursive source is not inherently erroneous:

- demanding its first element may converge;
- demanding a finite prefix may converge when each required production step converges;
- demanding a scalar that requires exhausting the source diverges when the source is proven infinite.

The compiler determines which computation the result actually demands and analyzes that computation. A separate user-visible productivity proof system is not currently required; productivity remains useful implementation terminology for the production of infinite structures.

Collections are not a special divergence category. Their convergence follows from recursion or corecursion, the operation's demand, and the convergence of required steps.

### Useful Proof Patterns

No single proof formalism is privileged. Useful sufficient convergence proofs include:

- an acyclic demanded computation graph whose required operations converge;
- finite structural recursion over a list, tree, or other finite structure;
- progress toward an exit under a well-founded measure or other invariant argument;
- finite traversal consisting of convergent steps;
- a finite demand on a corecursive source when each demanded production step converges.

Useful divergence proofs include:

- exact recursive repetition with no relevant exit or stopping mechanism;
- invariant-preserving progress away from the only exit condition;
- a closed state cycle that cannot reach a result;
- an operation whose semantics require exhausting data proven to be infinite.

These are examples rather than a complete algorithm. The implementation may use call graphs, dependency graphs, cycle detection, strongly connected components, invariant reasoning, or other techniques without exposing one of them as the source-language rule.

### Higher-Order Convergence

A higher-order operator is not automatically possibly divergent merely because it accepts another operator. Its definition may carry conditional convergence information.

For example, applying `F` twice can converge when both demanded applications of `F` converge. Reducing a finite collection with `F` can converge when the collection is finite and every demanded application converges. At a call site, those conditions are instantiated with the actual operator and arguments. Recursion or corecursion signals a need for analysis, not proof of divergence by itself.

## Pursuit Kinds

The essential type phrase is **a pursuit of T**. In categorical discussion, *pursuit* is the umbrella concept; the unqualified type phrase denotes the stoppable form, while the separate unstoppable monad is explicitly qualified. A pursuit is not synonymous with proven divergence: it may represent unresolved external activity, potentially divergent computation, a deliberately stoppable convergent computation, or a composition of other pursuits.

### Stoppable Pursuit

A **stoppable pursuit** guarantees an available mechanism that can eventually stop it. Native Thyrym logic normally establishes this through usable `yield` points. A killable external execution container can supply a different stopping mechanism.

Use *stoppable pursuit* when the categorical adjective is needed. “Ordinary pursuit” is not a technical category.

### Convergent Pursuit

A **convergent pursuit of T** is stoppable and additionally guarantees:

> If allowed to continue instead of being stopped, it eventually reaches `T`.

It remains a pursuit so a caller may stop even a finite computation that is expected to take a long time.

### Waylaid Convergent Pursuit

A **waylaid convergent pursuit of T** is a convergent pursuit whose ability to stop has escaped to asynchronous or otherwise independently surviving computation. It still reaches `T` if not stopped, but its current holder can no longer guarantee that an external participant will not stop it.

`waylaid` is provisional terminology. A plain convergent pursuit may weaken to this form when stop access escapes. Both forms satisfy stoppable-pursuit requirements, but composition retains the waylaid state while the external stop path remains possible.

Composing a waylaid convergent pursuit with another convergent pursuit through the stoppable monad remains waylaid. Composing with a merely stoppable, potentially divergent pursuit loses the combined convergence guarantee unless it can be proven again.

### Unstoppable Pursuit

An **unstoppable pursuit of T** has neither established natural convergence to `T` nor an established mechanism guaranteed to stop it. It remains a valid value that may be constructed, returned, stored, transformed, analyzed, or passed to operators. It cannot be treated as if guaranteed progress were available and cannot be awaited directly.

Stoppable and unstoppable pursuits are separate monads that do not implicitly bind through each other. In particular, placing `yield` before an unresolved unstoppable computation cannot launder that computation into a stoppable pursuit.

## Affine Control

Pursuits are affine because stopping and asynchronous control require unique stateful authority. Control remains represented by the pursuit's affine state and properties; there is no separate first-class stop-authority object.

Changing a convergent pursuit into a waylaid convergent pursuit is an affine mutation because the transformed pursuit survives. A final exclusive `await`, a synchronous stop that ends the relevant use, or killing a containing execution resource consumes the controlled resource because no surviving form is returned.

The decisive moment for waylaying is when future stop access escapes into independently surviving execution, not when a stop is eventually requested. An operation that creates this relationship must declare the affine mutation in its act or operator contract. A synchronous `stop P` does not waylay `P`; it initiates stopping and consumes or ends the relevant live use.

## `yield` And Stoppability

`yield` is the cooperative stopping checkpoint:

```thyrym
yield
```

It captures the next operator as its argument using the same continuation-capturing pattern as contextual and monadic bindings. If the continuation targets `T`, `yield` constructs the appropriate kind of pursuit targeting `T`.

Semantically, `yield` states that the surrounding pursuit may safely honor a pending stop request at that point under the invariants then established. It need not compile to a literal flag check; a compiler may use a flag, scheduler boundary, continuation, fused branch, no generated code when stopping is irrelevant, or another mechanism that preserves the contract.

### Stop-State Invariants

Each usable yield point contributes the invariants proven at that point. If a pursuit may stop at yield points with invariant conjunctions `I1`, `I2`, and `I3`, its stop state is an exclusive logical choice among those cases. The invariant solver may derive facts common to every case.

The exclusivity is logical. XOR is not a structural type-composition operator, as described in [types](types.md#invariants).

### Construction Rule

A `yield` does not make an arbitrary continuation stoppable. From each usable yield point, every potentially indefinite execution path must either return normally or reach another usable yield after only convergent work. No path may disappear forever into unresolved nonconvergent computation between stopping boundaries.

A valid indefinitely repeating shape is:

```text
yield
  -> convergent region
  -> yield
  -> convergent region
  -> ...
```

The whole computation may diverge while remaining stoppable. This shape is not established as stoppable:

```text
yield
  -> operation that may run forever without a stopping mechanism
  -> yield
```

It remains unstoppable unless the middle operation is proven or assumed convergent, or is protected by another valid stopping mechanism.

Convergence analysis asks whether the demanded computation eventually reaches its destination result. Stoppability analysis instead asks whether every indefinite path is guaranteed to encounter an available stopping mechanism. Stoppability may use convergence proofs for regions between yields.

Conceptually, when no stopping mechanism applies, proven convergence produces `T` and unresolved convergence produces an unstoppable pursuit of `T`, subject to normal contextual resolution. Around `yield`:

- a convergent continuation produces a convergent pursuit;
- a continuation whose indefinite paths always return to a yield after convergent work produces a stoppable pursuit;
- a continuation with a path that may run indefinitely before another valid mechanism produces an unstoppable pursuit.

## Stopping States

`stop` requests cooperative stopping. It does not mean immediate forced termination.

A pursuit moves conceptually through these states:

```text
running
  -> first stop request
stopping
  -> usable yield or successful stopping mechanism
stopped
```

`stopping` means the request has been accepted while execution continues toward a usable mechanism. `stopped` is the terminal outcome after that mechanism takes effect. Requests are race-safe and idempotent in effect; later requests while stopping do not change the semantic state.

Natural completion may beat a pending stop. A pursuit can receive a request, continue through convergent work, and return `T` before reaching another yield. The result is then `T`, not `stopped`. Race results must therefore permit several competitors to finish naturally.

`bypass` is distinct from `stop`. Bypassing lets one dependent path cease waiting while the underlying pursuit may continue; stopping requests that the pursuit itself end. The `or` combinator provides the same important non-stopping behavior by returning unresolved alternatives as live affine pursuits.

## Cooperative `stop` And Forced `kill`

`stop` follows the pursuit's semantic stopping contract and may wait for a suitable yield. It can preserve strong yield-point invariants but makes little general latency promise beyond eventual stoppability.

`kill` forcibly terminates an external execution container, such as a thread- or process-like chamber. It can provide more prompt termination but guarantees only the postconditions in the container's kill contract; it does not inherently preserve cooperative yield-point invariants.

> **`stop` favors semantic and invariant guarantees but may be slow. `kill` favors prompt forced termination but may expose weaker state guarantees.**

When source selects stop semantics, the compiler may implement them by killing a container only when the kill contract satisfies every required stop guarantee. Depending on stronger yield-point invariants can therefore change a previously fast implementation into cooperative stopping without changing source semantics.

A killable container may provide a weak stopping view, while contained code with valid yields may also provide a stronger cooperative view. Implicit conversion may select a view that satisfies the requested contract, but it cannot silently change the programmer's selected policy between `stop`, `kill`, or another mechanism.

## `await`

`await` accepts one convergent pursuit and consumes it. `and`, `or`, and `versus` are ordinary operators over pursuits that construct a new pursuit; they are not clauses in special `await` list grammar.

Normal precedence therefore interprets:

```thyrym
await P or Q
```

as awaiting the pursuit constructed by `P or Q`. Parentheses can override precedence.

Awaiting an exclusively controlled convergent pursuit of `T` consumes it and returns `T`. Consumption relinquishes the remaining ability to stop it; no intermediate “promised not to stop” type is needed.

Awaiting a waylaid convergent pursuit returns:

```text
stopped or T
```

The await itself converges: absent external stopping the pursuit reaches `T`, while an accepted external stop eventually reaches `stopped`. This is structural type `or`, not logical XOR.

An unstoppable pursuit cannot be awaited directly. It must first participate in a construction that establishes sufficient convergence and stoppability, such as containment in a killable asynchronous chamber combined with a convergent race condition.

### Implicit Destination Conversion

When a context requires `T` and holds an exclusive convergent pursuit of `T`, the compiler may perform an implicit consuming await-like conversion. Affinity ensures that no surviving use can later stop it. This can make a convergent finite demand on an infinite source available as an ordinary result.

A waylaid convergent pursuit cannot implicitly become `T` because the possible `stopped` result must be represented.

## Pursuit Composition

The pursuit operators have provisional precedence:

```text
and > versus > or
```

Thus `A and B versus C or D` groups as `((A and B) versus C) or D`. Parentheses use normal Thyrym syntax. No pursuit-specific grouping form is needed.

### `and`

For pursuits `P` of `PR` and `Q` of `QR`:

```thyrym
P and Q
```

constructs a pursuit whose successful target is the product:

```text
a PR and a QR
```

It resolves after every required constituent resolves and is convergent only if each required constituent converges. A required unstoppable constituent keeps the result in the separate unstoppable context rather than becoming implicitly stoppable.

### Inclusive `or`

`or` resolves once at least one constituent resolves and does not stop slower alternatives. For two constituents, the target states are structurally equivalent to:

```text
(PR and Q)
or
(P and QR)
or
(PR and QR)
```

The state `P and Q`, with both still running, is excluded at resolution. For N constituents, every nonempty combination contains each constituent as either its completed result or its still-running affine pursuit. Existing invariants can refine which combination occurred.

`P or Q` returns a pursuit of that combined state, not the state immediately. The combined pursuit converges when at least one constituent is guaranteed to converge. Consequently, a bounded convergent timeout such as the provisional:

```thyrym
wait 5 seconds
```

can make `P or wait 5 seconds` convergent even when `P` may diverge. If the timeout wins, `P` continues and appears as a live pursuit in the result.

### Competitive `versus`

`versus` is an n-ary race rather than timing-sensitive left-associated binary nesting:

```thyrym
P versus Q versus R
```

Its semantics are:

1. run all competitors;
2. after at least one resolves naturally, apply each unfinished competitor's configured brakes;
3. wait until every competitor reaches a terminal state;
4. resolve the combined pursuit.

There is no unique-winner guarantee. Competitors may finish simultaneously, before a stop request is issued, or after a request but before reaching a yield.

For `P` of `PR` and `Q` of `QR`, the target is structurally:

```text
(PR or stopped) and (QR or stopped)
```

with an invariant that at least one constituent has its normal result. Every original pursuit is represented by a terminal outcome because the race waits for all competitors and consumes their resources. This differs from `or`, which returns unresolved competitors alive.

A `versus` pursuit converges when at least one competitor converges to trigger resolution and every possible unfinished competitor has a braking mechanism that converges and guarantees termination. Brakes may use cooperative stop, forced kill, or a custom strategy satisfying the same contract.

## Braking Behavior

The provisional syntax for replacing a competitor's race-loser behavior is:

```thyrym
P braking with stop it
P braking with kill it
```

`braking with` is conceptual sugar for replacing a structural property provisionally named `Brakes`, not for introducing a nominal wrapper type. Different pursuit instances satisfying the same structural requirement may provide different implementations. The replacement must converge, guarantee that the losing pursuit is no longer running when it finishes, and expose its truthful terminal and post-state guarantees.

Replacing brakes preserves unrelated properties but replaces braking-related guarantees. `kill it` therefore does not retain yield-point invariants unless its kill contract proves them.

Without an override, `versus` uses the pursuit's cooperative stop behavior, conceptually as if every competitor said `braking with stop it`. The compiler may still use kill internally when its contract fulfills the selected stop semantics. Custom brakes may implement policies such as graceful shutdown followed by bounded waiting and forced termination.

`braking with`, `Brakes`, and their exact declarations are provisional.

## Asynchronous Containment

Independent execution is compiler-significant because the compiler must track affine values that outlive the current path and stop access that escapes to them. The provisional term **asynchronous chamber** denotes an affine, independently progressing execution resource. Ordinary pure code cannot manufacture one; a trusted foreign or system contract may declare that it creates one. Threads, processes, and tasks can be concrete forms.

A kill-like operation consumes a chamber and establishes that its execution has ended and cannot resume. Affine consumption by itself does not imply termination; the operation's contract supplies that guarantee.

A system operation may place an unstoppable pursuit inside a killable chamber and return stoppable outer execution:

```text
unstoppable pursuit of T
  -> spawn into a killable asynchronous chamber
  -> chamber-backed stoppable pursuit
```

The inner computation has not become cooperatively stoppable. The outer computation is stoppable because its chamber can be terminated.

Foreign contracts must state exactly what forced termination does to affected affine state. If termination weakens, invalidates, or consumes resources, the resulting types must expose only the state and capabilities that remain sound. Pursuit design does not introduce a broad “unsafe to terminate” category in advance of a concrete runtime need.

## Shared State At Stopping Boundaries

Stopping relies on the general invariant rule that a relationship among shared mutable values is externally guaranteed only when no observer can see a violating intermediate state. A semantic transition over one shared state object must preserve observable atomicity, whether implemented with private construction and reference replacement, locking, transactions, hardware atomics, or another equivalent mechanism.

Separate mutable resources do not automatically preserve cross-resource relationships. Cooperative yields are safe observable boundaries and must meet their promised invariants. Forced kill may use a weaker boundary and therefore expose weaker postconditions.

## Pursuits And Acts

Pursuits and acts are orthogonal monads. A pursuit describes whether completion is established and how ongoing computation can be stopped. An act describes interaction with external state and effects.

Pure convergent values, pure stoppable or unstoppable pursuits, acts producing ordinary values, acts producing pursuits, and effectful pursuit control are all meaningful. External work is not a pursuit merely because it is slow, and a pure mathematical computation may be a pursuit.

A value may become available while independent effectful work remains. Dependency, effect, and affine contracts determine what can proceed rather than procedural stack-frame completion. Pure unused pursuits may be eliminated, but an act cannot be eliminated merely because its return value is unused when required effects remain.

A terminating top-level program completes only after required top-level acts and outstanding effect obligations complete. Long-running servers may instead be modeled as intentionally indefinite stoppable pursuits whose stopping is triggered by an external shutdown event.

## Analysis And Scope Boundaries

Convergence and stoppability analysis should grow incrementally. Candidate implementation techniques include dependency graphs, recursive-component analysis, conditional higher-order summaries, invariant proofs, and treating yields or killable containers as stopping boundaries. These techniques explain the intended direction but are not mandatory source semantics.

Resource exhaustion, including memory, descriptor, hardware, and other finite-resource failure, is outside the pursuit model. Pursuits may help control runaway work, but they do not solve resource exhaustion generally.
