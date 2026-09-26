# Pursuit Decisions

This document records the reasoning behind the [pursuits](../design/pursuits.md#pursuits-convergence-and-stopping) design without extending its guarantees.

## Guarantees Instead Of Schedules

Evaluation strategy remains free because eagerness, laziness, memoization, parallelism, and compile-time execution are implementation choices when they preserve all observable contracts. Convergence and stoppability are semantic because they determine whether a demanded result or stopping boundary is guaranteed, not because they prescribe a schedule.

## Three-Way Convergence Analysis

Termination is undecidable, so treating every unproved case as divergent would reject useful programs, while treating it as convergent would hide nontermination. Proven, disproven, and unknown outcomes fit the existing invariant solver and assumption boundary without inventing a second trust mechanism.

Demand-sensitive analysis admits useful finite observations of infinite data. It also avoids a separate productivity system whose guarantees would substantially overlap convergence analysis.

Higher-order definitions retain conditional summaries because classifying every operator argument as a divergence risk would make reusable abstractions needlessly imprecise.

## Pursuits As Descriptive Values

An unstoppable pursuit is valid because its type tells the truth: neither completion nor stopping has been established. Prohibiting its construction would remove useful representations without improving soundness. Restricting operations that require progress, particularly `await`, enforces the guarantee at the point where it matters.

A convergent computation may remain a pursuit because convergence says that it eventually finishes if allowed to continue, not that waiting is cheap or that the caller has no reason to stop it.

## Separate Stoppable And Unstoppable Monads

Implicitly binding an unstoppable pursuit into a stoppable one would falsely promise access to a stop boundary. Keeping the monads separate prevents an outer `yield` from laundering an indefinitely blocked inner computation into a stronger guarantee.

## Cooperative Yield Rule

One yield before unknown work is insufficient because a pending request can never take effect if execution remains in that work forever. Requiring every indefinite path to return to a usable yield after convergent work gives stoppability its eventual-progress meaning while still permitting intentionally infinite loops.

Yield-point invariants provide meaningful safe states rather than merely implementation checkpoints. Logical exclusive choice captures multiple possible stopping states, while common facts remain derivable.

## Affine Stopping Authority

Stopping authority is kept in the affine pursuit state rather than split into a separate cancellation token. This lets existing mutation, consumption, and escape analysis express exclusive control and waylaying without another resource hierarchy.

Waylaying occurs when stop access escapes because that is when exclusive control is lost. Waiting until a later stop request would leave the type unsound during the intervening asynchronous lifetime.

## Stop And Kill

Cooperative stop and forced kill make different promises. Stop privileges safe invariant-bearing boundaries, while kill privileges prompt container termination and can only expose the weaker state its runtime contract supports. Keeping the policies distinct lets source select semantics while still allowing the compiler to implement stop with kill when all required guarantees coincide.

Natural completion can win after a stop request because cooperative stopping takes effect only at a suitable boundary. This is why competitive races cannot soundly promise one unique normal result.

## Simple `await`, Composable Pursuits

Making `and`, `or`, and `versus` operators over pursuits keeps `await` unary and reuses normal parsing, precedence, and monadic composition. A bespoke await-list grammar would duplicate language machinery and make pursuit expressions less composable.

Inclusive `or` preserves unfinished affine pursuits because its purpose is to continue when any branch resolves without implicitly choosing a stopping policy. `versus` is separate because racing and braking losers is materially different behavior.

`versus` waits for all competitors to become terminal so no consumed affine pursuit remains running invisibly. Its convergence consequently depends both on one racer finishing and on every possible loser's brakes finishing.

## Structural Brakes

A replaceable structural `Brakes` property permits cooperative stop, forced kill, and user-defined shutdown policies without hard-coding each policy into `versus`. Replacement must carry replacement guarantees; retaining stronger old invariants after selecting weaker brakes would be unsound.

## Compiler-Significant Asynchrony

Independent execution cannot be only an arbitrary library convention because the compiler must know when affine resources and stop access outlive the current path. A compiler-known asynchronous resource allows trusted FFI contracts to describe threads, processes, or tasks while leaving concrete runtime mechanisms outside the language core.

Container termination can make an outer computation stoppable without pretending that contained unstoppable logic gained cooperative yields. This preserves the distinction between logical and environmental stopping mechanisms.

## Effects Remain Orthogonal

Acts describe observable interaction; pursuits describe progress and stopping. Conflating them would classify pure long-running calculations as effects and externally slow but contractually convergent operations as pursuits without justification.

Top-level effect obligations outlive unused return values because eliminating required effects would change program meaning. Pure unused pursuits have no corresponding obligation and may be discarded.
