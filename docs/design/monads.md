# Monads And Contextual Binding

Thyrym preserves Haskell-style algebraic distinctions among functors, applicatives, monads, and lawful transformer behavior while reducing explicit transformer plumbing. Contextual structure is implicit only where a declared transformation is lawful and meaning-preserving; binding depth remains an explicit programmer choice.

The reasons behind these choices are recorded in [monads](../rationale/monads.md#monad-decisions). Declaration grammar and other unsettled details are tracked in [monads](../unresolved/monads.md#monad-questions).

## Type Phrases

The IO-like monad uses the phrase:

```thyrym
an act making A
```

`act` names an effectful computation as a first-class value, while `making` introduces its successful result type. The no-useful-value type is `nil`:

```thyrym
an act making nil
an act making an error or nil
```

Error sums use `or` directly rather than an additional `either` keyword:

```thyrym
an error or a string
```

Natural type phrases preserve real constructor nesting. These are distinct:

```thyrym
an act making an error or a string
an error or an act making a string
```

The first performs an act before an error result is known. The second is already either an error or an act.

## Functor, Applicative, And Monad

Functor mapping transforms a value inside one context. If the transformation itself returns the same context, mapping alone produces nested contexts rather than flattening them.

Applicative composition combines contextual computations whose structure is independent. Monad binding is required when a later computation's structure depends on an earlier result. Thyrym may infer the applicative case from dependencies, but this does not erase the semantic distinction.

`through` is the current candidate for functor mapping:

```thyrym
Users
    through Name of it
    through it trimmed
    unless it is empty
```

Here `it` is the implicit unary argument, and `unless` filters by the negation of its predicate. The general-purpose `through` syntax remains provisional.

## Lawful Transformer Behavior

Two monads `M` and `N` do not make `M (N A)` a monad automatically. Their sequencing semantics require an explicit interaction law.

A transformer can be understood conceptually as placing an outer context inside an underlying one:

```text
OuterT Inner A = Inner (Outer A)
```

Thyrym should allow one conceptual monad type to define both its ordinary behavior and any transformer or distribution behavior it lawfully supports. This does not make every monad a transformer. In particular, a naive `m (list of A)` representation does not provide a lawful general list transformer for every `m`; nondeterminism may require a more specialized encoding.

### Directional Distribution

Distribution laws are directional. An error context can lawfully move inside an act:

```text
an error or (an act making A)
-> an act making (an error or A)
```

An existing error becomes an act that makes that error, while the success constructor can be mapped over an existing act.

The reverse is not generally available:

```text
an act making (an error or A)
-/> an error or (an act making A)
```

Choosing the outer error branch would require running the act and observing its result. A pure expression cannot inspect that effect in advance. Acts are therefore not assumed to distribute through arbitrary monads.

Lists also do not silently move outside arbitrary effects. Mapping an effectful operation over a list naturally produces a list of contextual computations. Combining those computations into one contextual list requires an explicit sequencing or collection operation.

## Implicit Context Conversion

The general implicit conversion solver may perform only declared, meaning-preserving contextual conversions.

It may lift a plain value outward through available `pure` or `return` behavior:

```text
A
-> an error or A
-> an act making (an error or A)
```

The conversion from `an error` to `an error or A` is behavior specifically provided by the error monad, not a universal rule for every multi-parameter monad.

The solver may also apply an explicitly declared lawful distribution, such as moving an error context inside an act. It may not invent a distribution for arbitrary context pairs, silently unwrap a context, or discard its semantic information.

The solver's implementation remains unresolved.

## Explicit Caret Depth

Ordinary definitions use `:=`. A caret in a binding operator binds through exactly one contextual layer:

```thyrym
Value := expression
Value :^= expression
Value :^^= expression
Value :^^^= expression
```

The compiler never infers the number of carets. Expected types, applicative inference, implicit lifting, and lawful distribution cannot change the requested binding depth.

Given an expression of type:

```thyrym
an act making an error or a map of IDs to strings
```

this binding:

```thyrym
Usernames by ID :^= get all usernames
```

binds only the act. The name has type `an error or a map of IDs to strings`, allowing the program to inspect or recover from the error as data.

This binding:

```thyrym
Usernames by ID :^^= get all usernames
```

binds through both the act and the error. The name has type `a map of IDs to strings`, while the error branch participates in the enclosing monadic flow.

## Prefix Sequencing

Leading carets sequence contextual computations whose successful payload is not named. Each caret again binds exactly one layer:

```thyrym
^log "Starting..."
^^log "Searching..."
```

The prefix operation passes control to its continuation without naming the bound result. A final expression can omit the prefix caret and terminate the chain naturally.

For an operation producing `an act making an error or a list of strings`, `^^operation` sequences the act, propagates the error, and discards only the successful list payload. `^operation` sequences only the act and leaves the entire error context in the discarded result.

## Dangerous Discards

Danger is semantic rather than a consequence of being an applicative or monad. A context is dangerous when silently discarding it is likely to hide a program failure.

The current policy is:

| Context | Dangerous by default | Reason |
| --- | --- | --- |
| `an error or` | Yes | Discarding it can suppress failure. |
| maybe/perhaps | No | Absence is a valid result; an operation should use an error when absence means failure. |
| `a list of` | No | Discarding returned data does not inherently hide failure. |
| `an act making` | No | Running an act for effects while ignoring its successful payload is legitimate. |

The danger property belongs to the Applicative abstraction and defaults to non-dangerous. Because every monad is also applicative, user-defined monads and other applicatives have one place to opt into dangerous-discard checking. The exact declaration syntax and propagation through deeply nested contexts remain open.

If prefix sequencing stops before a dangerous layer, compilation fails unless the author explicitly acknowledges the discard with `!`:

```thyrym
^!log "Searching..."
```

`^!` binds exactly one layer and acknowledges dangerous context left in the discarded result. It does not bind an additional layer. More carets remain the preferred form when failure should propagate:

```thyrym
^^log "Searching..."
```

Explicit acknowledgement counts as accounting for the possible failure; unacknowledged discard remains invalid.

## Applicative Inference

Thyrym uses the same caret syntax for bindings that may be lowered applicatively. A separate applicative assignment operator is not needed.

```thyrym
Profile :^= load user profile
Settings :^= load user settings
Summary := summarize Profile Settings
```

If neither load depends on the other's result, the compiler may combine them applicatively. By contrast:

```thyrym
User :^= fetch current user
Settings :^= fetch settings of User
```

requires monadic sequencing because the second computation depends on `User`.

Dependency graphs may contain both forms. After fetching `User`, independent profile and settings loads may compose applicatively before a summary uses both.

Independent acts do not promise source-order execution. An implementation may reorder or run them concurrently when data dependencies and effect/resource contracts establish independence. The syntax and semantics for an explicit ordering boundary remain open.

## Combining Errors

`combining errors` is an ordinary typed operation rather than compiler behavior. Conceptually it transforms a list whose elements are errors or successful values into one error or a list of successful values:

```text
a list of errors or As
-> an error or a list of As
```

With no errors it returns the successful values. Otherwise it combines errors according to the declared error-combination semantics. This operation may eventually generalize to an appropriate finite collection abstraction, but it does not apply indiscriminately to every monad.

## Diagnostics

Because names need not carry explicit annotations, an insufficient caret count may become visible only at a later use. Diagnostics should trace that mismatch back when practical and suggest the likely binding depth. Dangerous-discard diagnostics should state which layers were bound and offer both safe propagation and explicit acknowledgement when applicable.
