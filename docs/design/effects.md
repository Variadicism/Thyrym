# Effects, Resources, And Trusted Interfaces

Thyrym represents observable interaction explicitly through typed acts and uses affine ownership for external resources. The rationale is recorded in [effects](../rationale/effects.md#effect-decisions), and unsettled details are tracked in [effects](../unresolved/effects.md#effect-questions).

## Acts

An **act** is a monadic representation of operations that may produce observable effects. Opening, reading, writing, or otherwise interacting with external systems commonly involves acts.

The IO-like type phrase is:

```thyrym
an act making A
```

Acts are explicit, but their types and contracts should expose effects more precisely than a single opaque IO category. This allows the compiler to determine which acts interfere.

When acts are proven independent, an implementation may potentially:

- reorder them;
- perform them in parallel;
- eliminate unnecessary work;
- duplicate them when their resource and effect semantics permit it.

Acts specify required effects and dependencies rather than one imperative schedule. Their monadic composition and binding syntax are described in [monads](monads.md#monads-and-contextual-binding).

## Affine Resources

External resources such as files are intended to use affine typing. Combined with precise act arguments and results, invariants, and explicit effect information, affine resources support static reasoning about:

- resource consumption;
- ownership and lifetime;
- interference;
- duplication;
- reordering;
- parallel execution;
- other effect-sensitive transformations.

The full affine-resource representation and lifetime model remain under design.

An affine **mutation** takes a resource and returns a transformed surviving form of that resource. **Consumption** returns no surviving form. Closing a file, killing an independently executing resource, and finally awaiting an exclusively controlled pursuit are examples of consumption; changing a pursuit's control state while returning the changed pursuit is mutation. Ordinary transformation-and-return should not be described as consumption.

## Independent Execution

Some resource contracts describe computation that progresses independently of the current path. Pursuit design provisionally calls such a compiler-significant resource an **asynchronous chamber**. It is affine, cannot be manufactured by ordinary pure code, and may be created by a trusted foreign or system operation. Threads, processes, and tasks may provide concrete forms.

An operation that consumes a chamber and establishes that its execution has ended is a kill-like operation. Consumption alone does not imply termination; the operation's contract must explicitly guarantee that the independent execution cannot continue or resume. Forced termination must also expose truthful post-state guarantees for every affected resource. Detailed containment and stopping semantics are described in [pursuits](pursuits.md#asynchronous-containment).

## Program Completion

A terminating program is complete only after its required top-level acts and outstanding effect obligations have completed. A pure unused computation, including an unused pursuit, may be discarded; an effectful obligation may not be discarded merely because no result uses its value. Value availability therefore does not by itself imply that all independent effects have finished.

## Foreign Functions

Thyrym expects to use foreign functions, especially C functions, for low-level and standard-library functionality.

Foreign declarations must provide detailed contracts describing behavior relevant to Thyrym's correctness model, potentially including:

- accepted inputs;
- possible outputs;
- failures;
- side effects;
- resource mutations;
- ownership and lifetime implications.

### Trusted Boundary

Thyrym cannot prove that arbitrary foreign code is memory-safe or obeys its declared contract. A foreign declaration therefore establishes an unsafe, trusted boundary.

Incorrect contracts or implementations can invalidate guarantees made by otherwise-safe Thyrym code. The language should not prohibit useful C techniques merely to make arbitrary foreign code statically provable.

Future compilation targets may require trusted interface mechanisms analogous to the C foreign-function boundary.
