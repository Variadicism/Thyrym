# Thyrym — Core Foundations

> **Status:** Early foundational documentation  
> This document describes the current high-level design of Thyrym. Some areas are intentionally incomplete and will be expanded from dedicated design discussions.

The reasons behind foundational choices are recorded in [core](../rationale/core.md#core-decisions). Provisional foundational areas and questions that still require design work are tracked in [core](../unresolved/core.md#core-questions).

---

# Part I — Language Overview

## 1. Overview

**Thyrym** is a general-purpose, pure functional programming language built around several closely related goals:

- express programs in a readable, English-inspired syntax;
- allow users to define new syntax through operators;
- use a powerful structural type system to represent not only what values are, but what is known to be true about them;
- eliminate language-level runtime errors by proving correctness requirements at compilation;
- represent side effects explicitly through monads and typed acts rather than allowing arbitrary mutation;
- preserve enough semantic information for the compiler to aggressively evaluate, specialize, reorder, and optimize code;
- keep the irreducible language core as small as practical.

Thyrym currently targets C, although the design may eventually allow other compilation targets.

Thyrym is built on a shared language engine called **Thyry**.

---

## 2. Thyry, Thyrym, and Aryya

Thyrym and Aryya are separate languages built on the same underlying engine.

### Thyry

**Thyry** provides shared language infrastructure, including:

- parsing;
- pattern matching;
- type resolution;
- mostly structural type matching;
- invariants;
- operator resolution;
- other compiler and interpreter machinery common to both languages.

Thyry should be thought of as the semantic and compiler infrastructure underneath the languages rather than as a user-facing language itself.

### Thyrym

Thyrym is a **general-purpose pure functional language**.

Its intended primary compilation target is C.

It permits ordinary runtime-sized data and general-purpose computation without the specialized restrictions found in Aryya.

### Aryya

Aryya is a more specialized language intended primarily for vectorized mathematical computation.

It shares substantial type-system and parsing machinery with Thyrym through Thyry, but has different syntax, semantics, constraints, and compilation targets.

Concepts should therefore be classified carefully:

- some belong to Thyry and naturally appear in both languages;
- some belong specifically to Thyrym;
- others belong specifically to Aryya.

Similarity between the languages should not be mistaken for identical language semantics.

---

## 3. Fundamental Design Goal: No Runtime Errors

One of Thyrym's primary goals is:

> **Valid Thyrym code should not encounter a language-level runtime error.**

This does **not** mean that the outside world cannot fail.

A requested file may not exist. A network service may be unavailable. An API may reject a request. These are legitimate runtime conditions outside Thyrym's control.

Thyrym's responsibility is to ensure that every such possibility is represented explicitly and accounted for by valid code.

By contrast, failures caused by invalid program state should be rejected during compilation.

Examples include:

- dereferencing an invalid value;
- accessing an array outside its proven bounds;
- violating an operator precondition;
- using a value without satisfying its required invariants;
- failing to account for an explicitly possible error;
- throwing an uncaught exception.

Thyrym has no ordinary exception-throwing mechanism.

Failures are values and may be returned by operators.

### Trusted boundaries

Some guarantees necessarily depend upon trusted information.

Two important examples are:

1. foreign-function contracts;
2. explicitly assumed invariants.

Incorrect trusted information can undermine Thyrym's safety guarantees.

Foreign code should therefore be understood as crossing an **unsafe/trusted boundary**. Thyrym does not attempt to prove arbitrary C code memory-safe.

This is analogous in spirit to an `unsafe` boundary in Rust: safe language semantics can provide strong guarantees even though explicitly trusted code can violate them.

---

## 4. Pure Functional Semantics

Thyrym is a pure functional language.

Ordinary computation is referentially transparent: replacing a pure expression with its value must not change the meaning of the program.

Mutation and observable interaction with the outside world are represented explicitly through typed monadic operations, particularly **acts**.

### Evaluation strategy is intentionally unspecified

Thyrym does **not** define whether ordinary computation is evaluated:

- strictly;
- lazily;
- partially at compilation;
- in parallel;
- using some mixture of these strategies.

A valid implementation may choose any evaluation strategy that preserves the semantics of the program.

The language specifies **what must be computed and what effects must occur**, not the schedule used to compute them.

This freedom is important to Thyrym's optimization model.

---

# Part II — Syntax and Operators

## 5. English-Inspired Syntax

Thyrym's syntax is heavily inspired by English.

For example:

```thyrym
open File to read and write

if Count of Args > 2,
    print "There are too many arguments!"
```

Values and variables are commonly written like proper nouns:

```thyrym
File
Count
Args
Pets
```

Other words form the grammatical structure of operators.

Symbols remain useful where they communicate operations naturally, particularly for arithmetic, comparison, definition, and monadic binding.

Thyrym does not attempt to literally be English prose. Instead, it tries to make code read like a concise technical dialect of English.

### Grammatical voice

Operator wording should generally reflect whether the operator represents a pure transformation or an effectful act.

Operations returning **acts** commonly use command-like verbs:

```thyrym
open File to read
close File
print Message
```

Pure transformations generally favor descriptive, relational, or dependent-clause phrasing:

```thyrym
Dog moved to Location
PreOp built using Context
Operator run with Arguments
```

This convention reflects Thyrym's immutable semantics. Pure operators describe what a resulting value **is**, while acts describe something that must **be done**.

The distinction is primarily a grammatical convention for readability rather than a separate semantic mechanism.

---

## 6. Operators

Thyrym has no separate category of ordinary functions.

> **Everything callable is an operator.**

An operator can have simple syntax resembling a conventional function or symbol, or it can define much richer grammar.

For example:

```thyrym
\'Player, an online player' teleported to 'Location, a location'
    => an act making a permissions error? :/=
```

might participate in syntax such as:

```thyrym
Variadicism teleported to (1000, 256, 1000)
```

Pure operators generally use descriptive rather than imperative phrasing, while operators returning acts may naturally use command forms.

The syntax before `=>` defines the operator's invocation pattern.

The expression after `=>` describes its return type.

---

## 7. Operator Declarations and Definitions

An operator may be **declared** without an implementation:

```thyrym
\'Player, an online player' teleported to 'Location, a location'
    => an act making a permissions error? :/=
```

The `:/=` ending can be read roughly as "not defined."

Declarations are used for:

- type requirements;
- forward declarations;
- contracts whose implementation exists elsewhere.

An operator may instead be **defined** using `:=`, followed by its implementation.

`:=` is also used by variable-definition syntax:

```thyrym
Pets := Cats + Dogs
```

This reflects a broader idea: variable definitions are themselves implemented through operators and contextual grammar rather than requiring a separate mutable storage model.

---

## 8. Context-Dependent Grammar

Thyrym's grammar is **not context-free**.

The operators available in the current scope determine what source code can be parsed and how it is understood.

Users can introduce new syntax by defining operators.

Operator declarations and definitions are stored in a contextual stack.

Resolution proceeds from the most recently introduced operators downward.

This makes local syntax and shadowing fundamental features of the language.

### Ordered overload resolution

The newest operator is not automatically chosen merely because its text looks appropriate.

A candidate must be applicable.

Resolution considers:

1. lexical/pattern compatibility;
2. argument types;
3. expected return type;
4. relevant type invariants and other requirements.

If a newer operator matches syntactically but its argument types do not match, resolution continues downward.

Likewise, if its return type does not satisfy the required return type, resolution may continue downward and find an older operator whose return type does.

The effective rule is:

> **The newest applicable operator wins.**

This combines properties normally associated with both shadowing and overloading.

An older operator is truly unreachable only when a newer operator covers every context in which the older operator could have applied.

---

## 9. Thyry Patterns

Thyry does not begin by tokenizing source into a conventional fixed token grammar.

Instead, it parses composable **patterns**.

Important pattern forms currently include the following.

### Literal patterns

Literal patterns perform exact textual matching.

Spaces and other characters may matter exactly as written.

### `and` patterns

An invisible concatenation of multiple patterns.

### `or` patterns

Alternatives analogous to `|` in regular expressions.

### Delimited patterns

One or more occurrences of a pattern separated by another pattern.

The delimiter may itself potentially be empty.

### Space patterns

Patterns that recognize appropriate whitespace or word/punctuation boundaries.

They allow operators to express human-readable spacing rules without requiring every boundary to be encoded as a raw literal.

### Character-class patterns

A match for one character from a specified set or range.

Character classes may be negated.

### Argument patterns

The most important pattern form.

An argument pattern parses **one operator having a required type**.

For example:

```thyrym
'Player, an online player'
```

captures an operator whose value satisfies `an online player`.

Argument matching is therefore both syntactic and type-aware.

Type checking is expensive, so implementations may attempt cheaper lexical matching first both for performance and for clearer diagnostics.

### Named text patterns

A named text pattern captures the literal source text matched by another pattern and passes that text to the operator as a string.

For example:

```thyrym
"Name"(([A-Z]\w*){ })
```

may capture a Thyrym-style capitalized name and make the literal name available to the implementation as `Name`.

This is useful when operators generate contextual grammar.

---

## 10. Argument Boundaries

An operator's fixed grammar helps delimit its arguments.

For example:

```thyrym
Variadicism teleported to (1000, 256, 1000)
```

contains a required literal `teleported to`.

That literal participates both in recognizing the operator and in constraining where the first argument ends.

Arguments are therefore not necessarily delimited by conventional commas or parentheses. Their surrounding operator pattern can establish their boundaries.

---

## 11. Pre-Ops and Contextual Construction

An operator may accept another operator as an argument.

Thyrym also has a more unusual concept: a **pre-op**.

A pre-op is source syntax that has not yet been fully parsed, type-checked, and compiled into an operator under its final operator context.

It can later be **built using** additional contextual operators.

Conceptually:

```thyrym
PreOp built using Context
```

means:

> Resolve this pre-op under a context extended by these operators.

This is one of Thyrym's most important metaprogramming mechanisms.

### Context-sensitive constructs

A construct such as `if` can use this machinery to introduce refined type knowledge into its branch.

If a condition proves that some value has a stricter type, the branch may be built using a temporary operator that shadows the previous representation of that value with the refined type.

Similarly, a loop can build its body using a generated operator representing its loop variable.

A domain-specific syntax can introduce operators that exist only within the appropriate context.

For example, an SVG construct could make a `g` element operator available only inside SVG content.

Outside that context, `g` would simply not be applicable syntax.

### `run with`

Normal operator syntax may be arbitrarily elaborate, so Thyrym also provides a syntax-independent means of applying an operator through its named arguments:

```thyrym
Operator run with Arguments
```

`run with` can therefore be understood as Thyrym's canonical operator-application primitive.

Both `built using` and `run with` follow the language's grammatical preference for descriptive phrasing in pure code.

---

## 12. Compile-Time Grammar Generation

Thyrym operators may generate additional operator grammar.

For example, a loop construct can introduce an operator whose name is taken from the loop's source syntax.

However:

> **All grammar must be derivable at compilation.**

Runtime values such as user input cannot create new grammar.

Compile-time-known values may participate freely in grammar generation, including values computed from other compile-time-known values.

This guarantees that even pre-ops and contextual syntax can always be completely resolved before generated code is emitted.

Thyrym's grammar is therefore:

- context-dependent;
- scoped;
- generative;

but **never runtime-dependent**.

---

## 13. Scope and Operator Chaining

Thyrym does not require a primitive indentation-based block model.

Chaining occurs only when an operator explicitly accepts another operator or pre-op as an argument.

For example, a variable definition is useful only for the code that follows it, so its operator can accept a continuation pre-op.

A conceptual variable-definition pattern is:

```thyrym
\"Name"(([A-Z]\w*){ }) \:= 'Value, a thing' 'Next, a pre-op' :/=
```

Its implementation can:

1. capture the name;
2. create a new nullary operator under that name that returns `Value`;
3. take `Next` built using that operator.

Thus:

```thyrym
Pets := Cats + Dogs
...
```

does not imply mutable storage.

It establishes a contextual operator named `Pets` for the continuation.

Scope follows naturally from the pre-op being built with that additional context.

### No implicit chaining

Adjacent expressions are not automatically chained merely because they appear next to each other.

Chaining exists only when the grammar of one operator explicitly consumes another operator or pre-op.

This keeps composition explicit in the language's operator model.

---

# Part III — Types and Structural Satisfaction

## 14. Types Are First-Class Values

Types in Thyrym are themselves values.

They can be:

- passed as arguments;
- returned by operators;
- stored in parameters;
- inspected or transformed during compilation.

Thyrym is structurally typed.

A type is primarily a collection of operator capabilities and invariants rather than a nominal declaration of ancestry.

---

## 15. `one` and Type Construction

Types are expressed using a placeholder called `one`.

For example:

```thyrym
an animal := a type with
    \'one' moved to 'Location, a location'
        => an error or one at Location :/=
```

`one` behaves somewhat like a type-level lambda parameter.

When a candidate is tested against `an animal`, the candidate replaces `one`.

If `a dog` is being tested, the requirement becomes conceptually:

```thyrym
\'Dog, a dog' moved to 'Location, a location'
    => an error or Dog at Location :/=
```

The type checker then determines whether such a capability exists in the relevant context.

---

## 16. Structural and Operator Satisfaction

Thyrym uses the term **satisfies** for structural type relationships.

For example:

> `a dog` satisfies `an animal`

means that the dog provides all capabilities and guarantees required by the animal type.

The dog may provide additional capabilities.

This avoids relying on potentially ambiguous terminology such as "broader" and "narrower."

A candidate type satisfies another type by satisfying its declared operator requirements and invariants.

### Operator satisfaction

A candidate operator satisfies a required operator if the candidate covers every use promised by the requirement.

There are three primary dimensions.

#### Pattern coverage

Every syntactic form accepted by the requirement must also be accepted by the candidate.

The candidate may accept additional forms.

For example, a candidate matching:

```text
(House|Home) of ...
```

can satisfy a requirement matching:

```text
House of ...
```

#### Argument types

For each argument:

> **The requirement's argument type must satisfy the candidate's argument type.**

This means the candidate must be capable of accepting everything the requirement promises callers may provide.

In conventional type-theory terminology, arguments are contravariant.

#### Return types

The reverse direction applies to results:

> **The candidate's return type must satisfy the requirement's return type.**

The candidate may therefore return something more specific than the requirement promises.

In conventional terminology, return types are covariant.

A useful summary is:

> A satisfying operator may accept more than required, but never less; it may return something more specific than promised, but never something outside the promise.

---

## 17. Declarations and Definitions Inside Types

A type's **declarations** create requirements.

A candidate must provide operators satisfying those declarations.

Definitions inside a type do **not** themselves create additional satisfaction requirements.

However, a definition may be used to satisfy a declaration.

The exact long-term role of type-provided definitions as defaults or reusable capabilities may be expanded in later documentation.

---

## 18. `a thing`

The most general type in Thyrym is:

```thyrym
a thing
```

All ordinary values ultimately satisfy `a thing`.

It represents the minimum useful structural guarantee about a value.

---

## 19. Parameters and Generic Operators

Thyrym's generic type system uses **parameters**.

For example:

```thyrym
for all E,
    \'First, a list of Es' + 'Last, a list of Es'
        => a list of Es
        where Length := Length of First + Length of Last
```

A parameter such as `E` establishes a relationship between argument types and the resulting return type.

Parameters are not intended to appear only in the arguments or only in the return type.

Their purpose is specifically to relate the two.

### Bounds

A parameter may declare a minimum type.

Any type matched against that parameter must satisfy the minimum bound.

### Multiple matches

If the same parameter matches multiple argument types, all of those matches are tracked.

The resulting parameter value is formed by combining the matched types with `or`.

For example:

```text
list of cats + list of cats
→ list of cats

list of cats + list of dogs
→ list of cats or dogs

list of dogs + list of animals
→ list of dogs or animals
```

Normal structural simplification may make some combinations equivalent to a simpler type.

If unrelated values are combined:

```text
list of dogs + list of cars
```

the useful common guarantees may collapse to little more than:

```text
a thing
```

That is valid behavior.

A type error should occur only later if the result is used somewhere requiring capabilities it cannot guarantee.

### Parameters are values

Inside the implementation of the generic operator, `E` itself may be referenced as a type value.

### Invariants participate normally

Invariants are part of types and therefore participate in parameter matching and `or` combinations automatically.

No separate generic-invariant system is necessary.

---

# Part IV — Invariants and Compile-Time Knowledge

## 20. Invariants

Invariants are an integral part of Thyrym's type system.

A type can carry propositions about its values.

The basic syntax uses `where`:

```thyrym
an integer where it < 0
```

Common cases may have syntactic sugar:

```thyrym
an integer < 0
```

A candidate type satisfies a required type only if it satisfies both:

1. its structural operator requirements;
2. its invariant requirements.

For example:

```thyrym
an integer where it < -10
```

satisfies:

```thyrym
an integer where it < 0
```

because the former invariant implies the latter.

### Scoped uniqueness

Uniqueness is not a nominal marker in Thyrym. It is a scoped guarantee arising from meaningful capabilities and invariants. The guarantee may be deterministic or generative, structural or relational, or probabilistic.

For example, a collection whose identifying field is unique can provide the stronger guarantee that filtering by equality on that field returns at most one member. Inserting an arbitrary runtime identifier into such a collection requires evidence that the identifier is absent. A condition can establish that fact at runtime and refine the types available in its guarded branch, where the compiler can then prove that insertion preserves the collection invariant.

Injective operations transport the relevant uniqueness guarantee without changing its source. In particular, an injective transformation preserves a probabilistic collision guarantee but does not turn it into certainty.

The full model, including probabilistic identifier storage and partial identifiers, is described in [uniqueness](uniqueness.md#scoped-uniqueness). Its design reasoning is recorded in [uniqueness](../rationale/uniqueness.md#uniqueness-decisions), and unsettled details are tracked in [uniqueness](../unresolved/uniqueness.md#uniqueness-questions).

---

## 21. Invariant Reasoning

The invariant solver is not expected to be a complete theorem prover.

A required invariant may be:

- proven;
- disproven;
- unknown.

Ordinary type satisfaction requires proof.

An unknown result is therefore insufficient unless the programmer explicitly crosses a trust boundary.

The invariant system can reason about different interpretations of the same bits.

For example, less-than or equality may have different semantics for:

- unsigned integers;
- signed integers;
- floating-point values;
- other interpreted data.

The exact solver design remains under development.

---

## 22. Trusted Invariant Assumptions

Some mathematically valid guarantees may be impractical for the compiler to prove.

Numerical approximations are a common example.

A vectorized sine approximation may involve a complicated mixture of multiplication, division, powers, addition, subtraction, and floating-point behavior whose final invariant is difficult for a practical solver to establish automatically.

Thyrym therefore needs an explicit trusted assumption mechanism.

The preferred syntax direction is:

```thyrym
assuming Invariant
    Next
```

Semantically:

1. Thyry attempts to disprove the invariant.
2. If the invariant is disproven, compilation fails.
3. If it is proven, it is accepted normally.
4. If it remains unknown, the programmer's trusted assumption is accepted.
5. The continuation is built in a context enriched by the assumed invariant.

Thus:

> Thyrym may allow uncertainty to cross a trust boundary, but not a known falsehood.

Trusted assumptions become part of the trusted computing base behind Thyrym's safety guarantees.

---

## 23. `bits`: The Primitive Data Type

The only primitive type that directly represents data is:

```thyrym
bits
```

Other familiar data concepts are constructed from bits plus type information, invariants, and operators.

This includes concepts such as:

- integers;
- floating-point numbers;
- strings;
- records;
- collections;
- domain-specific values.

The same underlying bit representation may support different interpretations and therefore different operator and invariant semantics.

`bits` is necessarily part of Thyrym's irreducible core because no other data-bearing type can be expressed without it.

---

## 24. Static Values

The current preferred term for values guaranteed to be known during compilation is:

```text
static
```

Although `static` has several meanings in other languages, Thyrym should define it narrowly.

> **A static value is a value whose complete value is available during compilation.**

A known equality implies static availability:

```text
X = 7
```

means the compiler necessarily knows `X`.

However, static availability does not necessarily expose the actual value as part of a public type contract.

That distinction is useful when callers may rely on compile-time availability without being coupled to a particular constant.

For example, a logging format may be guaranteed static even if library users should not rely on its current textual contents.

### Propagation

If:

- an operator is referentially transparent;
- all of its arguments are static;

then its result is also static.

This provides natural recursive compile-time evaluation without requiring a separate metaprogramming language.

Acts depending on runtime or external state generally break static propagation.

---

# Part V — Monads, Errors, and Effects

## 25. Monads

Thyrym has explicit monads heavily inspired by Haskell.

Users may define their own monads.

Monadic composition follows essentially the same semantic ideas as Haskell, expressed through Thyrym's own syntax and operator system.

Functor mapping transforms values without flattening a returned context. Applicative composition combines independent contextual computations. Monad binding is required when the structure of a later computation depends on an earlier result.

Monads do not compose automatically. A monad may declare lawful transformer or distribution behavior for compatible inner contexts, but being a monad alone does not imply such behavior.

The type phrase for the IO-like monad is `an act making A`. Nested phrases preserve actual constructor order, so `an act making an error or A` and `an error or an act making A` are different types.

The full composition, binding, inference, and discard model is described in [monads](monads.md#monads-and-contextual-binding). Its rationale is recorded in [monads](../rationale/monads.md#monad-decisions), and unsettled details are tracked in [monads](../unresolved/monads.md#monad-questions).

---

## 26. Errors as a Monad

A type such as:

```thyrym
an error or Value
```

behaves like Haskell's `Either Error Value`.

It contains one of two possible values.

When bound monadically:

- a success/right value is passed to the continuation;
- an error/left value skips the bound continuation and propagates outward.

This allows multiple fallible operations to be written sequentially while automatically returning the first error.

Errors are therefore ordinary values.

They are not exceptions.

The no-useful-value type is `nil`, allowing types such as `an act making nil` and `an act making an error or nil`.

`an error or` is a dangerous context to discard because doing so can hide failure. It must be propagated, handled, retained as data, or explicitly acknowledged as discarded.

---

## 27. Monadic Binding Syntax

Thyrym currently uses `^`-based syntax for monadic binding.

Ordinary definition uses `:=`. Each caret in a binding operator explicitly binds through exactly one contextual layer:

```thyrym
Value := expression
Value :^= expression
Value :^^= expression
Value :^^^= expression
```

The compiler never infers caret depth. Fewer carets intentionally retain inner contexts as values. For example, if an expression has type:

```thyrym
an act making an error or a map of IDs to strings
```

then `:^=` binds only the act and names an `an error or a map of IDs to strings`, while `:^^=` also binds through the error and names the map. Implicit lifting or lawful distribution may occur around this explicit choice but cannot replace it.

Monadic binding syntax is expected to be implementable through the ordinary operator system rather than requiring primitive compiler support.

---

## 28. Monadic Chain Boundaries

A monadic chain does not depend on indentation. For a contextual computation whose returned value is not needed, each leading caret sequences through one layer without naming its successful payload:

```thyrym
^close File
^^log Message
```

The final expression in the continuation can omit a sequencing caret, providing a natural chain boundary.

Caret depth remains semantically significant for effect-only statements. Given an operation producing `an act making an error or nil`, `^^operation` binds through both the act and error so failure propagates. `^operation` binds only the act and would otherwise discard the remaining error context.

Dangerous contexts cannot be silently discarded. A trailing `!` after the carets explicitly acknowledges the discard:

```thyrym
^!log Message
```

The carets still choose binding depth; `!` only acknowledges dangerous information left in the discarded result. `an error or` is dangerous. Acts, lists, and maybe/perhaps contexts are not dangerous merely because they are contextual, although a dangerous context nested inside them may require handling under rules that remain under design.

The compiler may lower independent caret bindings applicatively rather than forcing monadic sequencing. It may not infer a different caret count. Dependent bindings still require monadic sequencing, and acts may be reordered or run concurrently only when their data, effect, and resource contracts establish independence.

Detailed rules and examples are provided in [monads](monads.md#explicit-caret-depth).

---

## 29. Acts

An **act** is a monadic representation of operations that may produce observable effects.

Opening, reading, writing, or otherwise interacting with external systems commonly involves acts.

Acts are explicit.

However, unlike a coarse IO system in which all effects become opaque, Thyrym intends act types and contracts to state their effects with significant precision.

This should allow the compiler to reason about which acts interfere with one another.

If two acts are proven independent, an implementation may potentially:

- reorder them;
- perform them in parallel;
- eliminate unnecessary work;
- duplicate them when their resource/effect semantics permit it.

Acts therefore define required effects and dependencies, not a fixed imperative execution order.

Detailed act/resource semantics, including affine resources, are covered in separate design discussions and are intentionally deferred from this foundational document.

---

## 30. Affine Resources

Thyrym intends to use affine typing for external resources such as files.

Combined with:

- precise act argument/return types;
- invariants;
- explicit effect information;

the system should allow static reasoning about:

- resource consumption;
- interference;
- duplication;
- reordering;
- parallel execution;
- other effect-sensitive transformations.

The full affine-resource design is not reproduced here and should be integrated later from its dedicated design documentation.

---

# Part VI — Program Organization

## 31. Imports

The import system is still provisional.

The current model treats an imported file roughly as an exportable operator context.

Only the context established by that file itself should be imported.

Operators that were merely imported by that file should not automatically leak transitively.

### Whole-file import

The intended syntax is approximately:

```thyrym
using "file.thy"
```

### Importing selected operators

```thyrym
using Format from "logging.thy"
```

Multiple operators use ordinary Thyrym list syntax:

```thyrym
using Format and Level from "logging.thy"
```

### Paths are values

Import paths should not use a special filename grammar.

They should be ordinary string expressions that:

- are static;
- satisfy the required file-path constraints.

This allows compile-time path construction:

```thyrym
Directory := "lib/"
using Format from Directory + "logging.thy"
```

provided the resulting path is static and valid.

### Local imports

Imports may be scoped locally.

Conceptually, `using ...` is an operator that:

1. loads an imported operator context;
2. consumes a following pre-op;
3. builds that pre-op using the augmented context.

### Import order

Import order matters.

Later imported applicable operators receive normal higher stack priority.

If one explicitly imported operator is obscured by a later import, a compiler warning would likely be useful.

A user can resolve such conflicts by:

- reordering imports;
- importing only selected operators;
- reducing an import to a narrower scope.

### Visibility remains unresolved

The exact export/private model is not yet decided.

Ideally, operators needed only within a local scope should simply be defined in that scope rather than requiring explicit visibility modifiers.

The difficult unresolved case is a helper shared by several top-level exported operators while not itself intended for export.

This should remain an open design area rather than being prematurely fixed.

---

## 32. Recursion

Recursion is fundamental in Thyrym.

Because the language is immutable and purely functional, recursive formulations replace many tasks that would be expressed through mutable iteration in imperative languages.

### Direct recursion

An operator may refer to itself within its own implementation.

Once its pattern/signature has been established, that operator can be available while its implementation is built.

### Mutual recursion

Mutual recursion requires forward references.

The existing declaration syntax:

```thyrym
... :/=
```

will be used for explicit forward declarations.

However, the eventual syntax for attaching a later implementation to an already-declared operator is unresolved.

Repeating an entire complicated operator pattern would be undesirable and error-prone.

A later mechanism should ideally refer to the identity of the existing declaration rather than requiring its complete syntax to be duplicated.

---

## 33. Conventional and `deviant` Operators

Thyrym permits extremely flexible operator syntax.

However, most operators are expected to follow a relatively small number of recognizable grammatical shapes.

Examples might eventually include forms resembling:

```text
X + Y
foo X
Length of X
X moved to Y
```

The exact families remain to be designed.

The current proposal is that operators following approved conventional forms need no special marker.

Operators using substantially unusual grammar would be explicitly marked:

```text
deviant
```

The purpose is not to restrict expressive syntax, but to establish a readable default dialect.

Users remain free to create specialized syntax where it provides genuine value, such as embedded HTML-like structures.

### Forward-reference benefit

Conventional operator shapes may be recognizable before their definitions are encountered.

This could allow the compiler to implicitly recognize forward uses of ordinary non-deviant operators.

Deviant operators, whose grammar cannot safely be inferred, would require explicit forward declarations before forward use.

The exact conventional forms and compiler algorithm remain open.

---

# Part VII — Trusted Interfaces and the Language Core

## 34. Foreign Functions

Thyrym expects to rely heavily on foreign functions, especially C functions, for low-level and standard-library functionality.

Foreign declarations must provide detailed and accurate contracts.

Relevant contracts may describe:

- accepted inputs;
- possible outputs;
- failures;
- side effects;
- resource mutations;
- ownership/lifetime implications;
- other behavior relevant to Thyrym's correctness model.

The detailed foreign-interface design has been explored separately and should be integrated from those dedicated discussions.

### Foreign code is trusted/unsafe

Thyrym cannot guarantee that arbitrary foreign code is actually memory-safe or obeys its declared contract.

A foreign declaration therefore establishes a trusted boundary.

Incorrect foreign contracts or incorrect foreign implementations can invalidate guarantees made by otherwise-safe Thyrym code.

Thyrym should not attempt to prohibit useful C techniques merely to make arbitrary foreign code statically provable.

---

## 35. The Irreducible Core

Thyrym is intended to have a very small built-in core.

The exact list has deliberately not been finalized.

The implementation process may reveal which pieces truly must be primitive.

Current likely core elements include:

- the `bits` type;
- operator declaration and definition machinery;
- Thyry's underlying pattern facilities;
- `built using`;
- `run with`;
- enough type machinery to bootstrap user-defined types;
- whatever facilities are necessary to report compilation failure.

Many constructs that might be compiler primitives in other languages are intended to be implementable as operators instead.

Possible examples include:

- conditionals;
- loops;
- variable bindings;
- monadic binding syntax;
- contextual DSL constructs;
- imports.

The exact bootstrap inventory should remain explicitly provisional until implementation experience demonstrates what must truly be intrinsic.

Future compilation targets may also introduce new trusted interface mechanisms analogous to the current foreign-function boundary.

---

# Part VIII — Deferred and Open Design Areas

## 36. Areas Intentionally Deferred

Several important systems have dedicated design discussions and should be integrated later rather than reconstructed incompletely here.

These include:

- the full affine-resource model;
- detailed act/effect contracts;
- foreign-function contract syntax and semantics;
- resource lifetime rules;
- some invariant-solver implementation details;
- specialized external-resource types;
- other compiler and optimizer details.

This foundational document should provide integration points for those summaries without duplicating or guessing their contents.

---

## 37. Open Design Questions

The following areas remain intentionally unresolved or provisional. The questions to be answered and the constraints on their answers are tracked in [core](../unresolved/core.md#core-questions).

### Operator core

The exact irreducible set of built-in operators and primitives is not final.

Implementation work should guide this decision.

### Conventional operator forms

The formal set of non-deviant operator shapes has not yet been established.

These shapes must be chosen carefully enough that forward syntax can be recognized reliably.

### Completing forward declarations

The syntax for attaching an implementation to a previously declared operator without repeating its full grammar remains undecided.

### Import visibility

Thyrym does not yet have a final solution for module-level helper operators that need to be shared by several exported operators without themselves being exported.

Explicit visibility modifiers are preferably avoided if scope can provide a cleaner solution.

### Import/module details

Separate compilation, compiled-module metadata, and the precise representation of exportable operator contexts remain future work.

### Invariant solver

The invariant system has a working conceptual foundation but is not fully designed.

Particularly complex mathematical normalization and implication rules remain an active area of development.

### Assumption syntax

`assuming ...` is the current preferred direction.

Exact syntax and surrounding operator form may still evolve.

### Static syntax

`static` is the preferred term for compile-time-known values, but its exact type syntax has not yet been finalized.

### Resource representation

The system likely needs a specialized built-in concept for affine external resources, but its precise type representation is deferred to the existing resource-design work.

### Uniqueness details

The exact declarations for scoped uniqueness, probabilistic generator metadata and collision policy, partial-identifier contract, composition rules, and injectivity proofs remain open. These items are tracked in [uniqueness](../unresolved/uniqueness.md#uniqueness-questions).

### Monad declarations and inference

The exact declarations for functors, applicatives, monads, lawful distribution, and dangerous contexts remain open, as do the implicit solver, deeply nested danger detection, pattern matching, law verification, and explicit sequencing boundaries. These items are tracked in [monads](../unresolved/monads.md#monad-questions).

---

# Part IX — Core Mental Model

## 38. Core Mental Model

A useful way to think about Thyrym is:

> **Thyrym is a tiny operator-based functional language in which grammar, types, and contextual knowledge are all extensible through the same underlying machinery.**

Operators do more than compute values.

They can:

- define grammar;
- accept other operators;
- accept unresolved pre-ops;
- introduce new operators into a continuation;
- refine the types visible to later code;
- generate contextual DSLs;
- establish generic relationships through parameters;
- represent pure transformations;
- represent explicit side effects through monads and acts.

Types are not nominal labels attached to values.

They are first-class values describing structural capabilities and invariants.

A value satisfies a type when the compiler can prove that it provides everything that type promises.

The language attempts to preserve those semantics for as long as possible so that correctness checking and optimization are two consequences of the same rich information.

Thyrym's central aspiration is therefore not merely convenient functional programming or unusual syntax.

It is to make enough of a program's meaning explicit that:

> **invalid program states can be rejected at compilation, while valid programs leave the implementation broad freedom in how they are executed.**
