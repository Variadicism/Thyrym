# Pursuit Questions

This document tracks details left unresolved by the [pursuits](../design/pursuits.md#pursuits-convergence-and-stopping) design. Statements here are not language guarantees until their answers are incorporated into the design.

## Provisional Terminology

Should `waylaid`, `braking with`, `Brakes`, and `asynchronous chamber` become the final source and implementation terms?

Their current meanings are settled enough to describe the model, but their names remain provisional. `stoppable pursuit`, `yield`, `stop`, `stopping`, `stopped`, `kill`, `await`, `and`, `or`, `versus`, and `bypass` are the current non-provisional vocabulary.

## Type-Directed Yield Selection

Should a caller be able to require stronger stop-state invariants that disable yield points failing those invariants?

Such a feature would need to prove that every remaining indefinite path still reaches an eligible yield after convergent work. It must also define behavior across branching, concurrency, and composed pursuits. Until then, library operators can accept ordinary options controlling which yield points are active.

## Concrete Convergence Analyses

Which convergence and divergence proof strategies should the first implementations support, and how should conditional higher-order summaries be represented?

The proof repertoire is deliberately open-ended rather than a blocking language-design question. Unknown cases remain unknown and may use the existing assumption boundary.

## Surface Declarations And Precedence

What exact declarations define pursuit types, the structural `Brakes` property, custom braking operations, and the provisional `and > versus > or` precedence?

The semantics are settled, but their full expression through Thyrym's still-evolving operator and type declaration grammar is not.

## Time APIs

What exact standard-library operation constructs a bounded convergent timeout pursuit?

`wait 5 seconds` is an illustrative direction, not finalized time API syntax. Timeouts are ordinary convergent pursuits rather than special race syntax.

## Asynchronous Runtime Integration

How do concrete thread, process, and task APIs instantiate asynchronous chambers, and what FFI contracts describe their creation, completion, and forced termination?

The contracts must expose truthful ownership and post-termination state. Exact runtime mappings remain platform-specific.

## Signals And Server Shutdown

How should operating-system signals, interrupts, and shutdown requests map to Thyrym stopping behavior and standard-library server APIs?

Servers can be modeled as stoppable pursuits, but no exact runtime mapping has been selected.
