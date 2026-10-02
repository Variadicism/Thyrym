# Operators And Grammar

Thyrym uses one extensible operator model for callable behavior, grammar, declarations, binding, and contextual construction. The rationale for these choices is recorded in [operators](../rationale/operators.md#operator-decisions), and unsettled details are tracked in [operators](../unresolved/operators.md#operator-questions).

## English-Inspired Syntax

Thyrym's syntax is inspired by English without attempting to be literal prose:

```thyrym
open File to read and write

if Count of Args > 2,
    print "There are too many arguments!"
```

Values and variables commonly resemble proper nouns. Other words form operator grammar, while symbols remain useful for arithmetic, comparison, definition, and monadic binding.

Operator wording generally reflects its role. Acts commonly use commands:

```thyrym
open File to read
close File
print Message
```

Pure transformations favor descriptive or relational wording:

```thyrym
Dog moved to Location
PreOp built using Context
Operator run with Arguments
```

This is a readability convention rather than a separate semantic mechanism.

## Operators

Thyrym has no separate category of ordinary functions: everything callable is an operator. An operator may resemble a conventional function or symbol, or define richer grammar.

```thyrym
\'Player, an online player' teleported to 'Location, a location'
    => an act making a permissions error? :/=
```

This declaration can participate in syntax such as:

```thyrym
Variadicism teleported to (1000, 256, 1000)
```

The syntax before `=>` defines the invocation pattern. The expression after `=>` describes the return type.

### Declarations And Definitions

An operator ending in `:/=` is declared without an implementation. Declarations support type requirements, forward references, and contracts implemented elsewhere.

An operator is defined using `:=`, followed by its implementation. `:=` is also used by variable-definition syntax:

```thyrym
Pets := Cats + Dogs
```

Variable definitions are operators that establish contextual grammar; they do not introduce mutable storage.

### Static And Property Operators

A **static op** is an ordinary operator whose implementation is selected directly by contextual resolution. A **property op** is an operator requirement declared by a type whose selected implementation is carried as evidence by a typed value. **Static op** is useful but provisional terminology.

Property implementations are ordinary definitions rather than a separate declaration category. A static definition may be selected to satisfy a property and later be invoked dynamically through the resulting evidence. Property requirements and carried evidence are described in [types](types.md#properties-and-carried-evidence).

## Context-Dependent Grammar

Thyrym's grammar is not context-free. Operators available in the current scope determine what source can be parsed and how it is understood. Declarations and definitions form a contextual stack searched from newest to oldest.

A candidate is applicable only when all relevant constraints hold:

1. Its lexical pattern is compatible.
2. Its argument types match.
3. Its return type satisfies the expected type.
4. Its invariants and other requirements hold.

If a newer syntactic match fails a type or invariant requirement, resolution continues to older candidates.

> **The newest applicable operator wins.**

An older operator is unreachable only when a newer operator covers every context in which the older one could apply.

Static declarations and property declarations inhabit this same stack and compete through the same applicability rules. A newer static declaration may therefore shadow a property declaration. Their difference matters after selection: a static call names the selected definition directly, while a selected property declaration invokes the implementation carried by the relevant type evidence.

### Return-Type Resolution

Return types participate in applicability rather than being checked only after an overload is selected. If the surrounding expression requires a result type, a candidate whose every possible result does not satisfy that requirement is skipped.

For declarations such as:

```thyrym
\interpret 'Text, a string'
    => an integer :/=

\interpret 'Text, a string'
    => a date :/=
```

an integer-typed use rejects the date-returning candidate:

```thyrym
Count, an integer := interpret Input
```

Return invariants participate fully. A use requiring `an integer where it >= 0` accepts only a candidate whose possible results establish that invariant.

When no surrounding return requirement exists, no return-type filter is invented. In `X := interpret Input`, the newest otherwise-applicable declaration is selected and its exact return type becomes the type of `X`.

### Provisional Resolution And Backtracking

Resolution may remain provisional while later source is parsed and checked. A locally applicable candidate can make the remaining program impossible to type-check; the compiler may then backtrack and continue with older declarations from the same lexical context.

Semantically, resolution chooses the newest available interpretation that permits a valid interpretation of the program. This may require examining substantial later code, but declarations introduced later do not retroactively enter an earlier lexical context.

Implementations may optimize search and commitment aggressively as long as they preserve these semantics.

### Shadowing Diagnostics

Contextual shadowing is intentional and supports local behavioral overrides. Accidental shadowing should nevertheless produce a warning, especially when imports introduce the competing declarations. Dead-code or unused-operator diagnostics should also identify near-miss overrides whose signatures prevent selection.

An explicit selective import can communicate intentional replacement. General warning-suppression syntax and an `explain resolution` diagnostic remain unsettled tooling concerns.

## Thyry Patterns

Thyry parses composable patterns rather than first tokenizing source under one fixed token grammar.

### Literal Patterns

Literal patterns match exact text. Spaces and other characters may be significant.

### `and` Patterns

An `and` pattern invisibly concatenates multiple patterns.

### `or` Patterns

An `or` pattern supplies alternatives analogous to `|` in regular expressions.

### Delimited Patterns

A delimited pattern recognizes one or more occurrences separated by another pattern. The delimiter may itself potentially be empty.

### Space Patterns

Space patterns recognize appropriate whitespace or word and punctuation boundaries without requiring every boundary to be a raw literal.

### Character-Class Patterns

A character-class pattern matches one character from a set or range and may be negated.

### Argument Patterns

An argument pattern parses one operator having a required type:

```thyrym
'Player, an online player'
```

This captures an operator whose value satisfies `an online player`. Argument matching is therefore both syntactic and type-aware. Implementations may attempt cheaper lexical matching before expensive type checking for performance and clearer diagnostics.

### Named Text Patterns

A named text pattern captures the literal source matched by another pattern and passes it to the operator as a string:

```thyrym
"Name"(([A-Z]\w*){ })
```

This can capture a Thyrym-style capitalized name for use when generating contextual grammar.

## Argument Boundaries

Fixed operator grammar helps delimit arguments. In:

```thyrym
Variadicism teleported to (1000, 256, 1000)
```

the required literal `teleported to` both identifies the operator and constrains where its first argument ends. Arguments therefore need not rely on conventional commas or parentheses.

## Pre-Ops And Contextual Construction

An operator may accept another operator as an argument. It may also accept a **pre-op**: source syntax not yet fully parsed, type-checked, and compiled under its final operator context.

```thyrym
PreOp built using Context
```

This resolves the pre-op under a context extended by the supplied operators.

Context-sensitive constructs use this mechanism to introduce temporary knowledge or grammar. A conditional can shadow a value with a refined type inside its branch. A loop can introduce its loop variable. A domain-specific construct can expose operators only within its own body.

### `run with`

Because ordinary operator syntax may be elaborate, `run with` provides syntax-independent application through named arguments:

```thyrym
Operator run with Arguments
```

`run with` is Thyrym's canonical operator-application primitive.

## Compile-Time Grammar Generation

Operators may generate additional grammar, including names computed from compile-time-known values.

> **All grammar must be derivable at compilation.**

Runtime input cannot create grammar. This ensures that every pre-op and contextual construct can be completely resolved before generated code is emitted. Grammar is context-dependent, scoped, and generative, but never runtime-dependent.

## Scope And Operator Chaining

Thyrym does not require a primitive indentation-based block model. Chaining occurs only when an operator explicitly accepts another operator or pre-op.

A conceptual variable-definition pattern is:

```thyrym
\"Name"(([A-Z]\w*){ }) \:= 'Value, a thing' 'Next, a pre-op' :/=
```

Its implementation captures the name, creates a nullary operator returning `Value`, and builds `Next` using that operator. Thus:

```thyrym
Pets := Cats + Dogs
...
```

establishes a contextual operator named `Pets` for the continuation. Adjacent expressions are not implicitly chained merely because they appear next to each other.

## Recursion

Recursion is fundamental in Thyrym and replaces many mutable iteration patterns.

An operator may refer to itself once its pattern and signature have been established. The operator can be available while its implementation is built.

Mutual recursion uses explicit forward declarations ending in `:/=`. The syntax for attaching a later implementation to an existing declaration remains unresolved; it should identify the declaration without repeating a complicated pattern.

## Conventional And `deviant` Operators

Most operators are expected to follow a small set of recognizable grammatical shapes, such as:

```text
X + Y
foo X
Length of X
X moved to Y
```

The proposed design requires no marker for approved conventional forms. Substantially unusual grammar is explicitly marked `deviant`.

This preserves specialized syntax while defining a readable default dialect. Conventional shapes may also support forward use because the compiler can recognize their grammar before seeing a definition. Deviant operators require an explicit declaration before forward use. The exact conventional forms and recognition algorithm remain open.
