# Types And Compile-Time Knowledge

Thyrym uses first-class structural types enriched with invariants. The rationale for this model is recorded in [types](../rationale/types.md#type-decisions), and unsettled details are tracked in [types](../unresolved/types.md#type-questions).

## Types Are First-Class Values

Types can be passed as arguments, returned by operators, stored in parameters, and inspected or transformed during compilation.

A type primarily describes operator capabilities and invariants rather than nominal ancestry.

## `one` And Type Construction

Types use a placeholder called `one`:

```thyrym
an animal := a type with
    \'one' moved to 'Location, a location'
        => an error or one at Location :/=
```

`one` behaves like a type-level parameter. When testing `a dog` against `an animal`, the candidate replaces `one`, producing a conceptual requirement such as:

```thyrym
\'Dog, a dog' moved to 'Location, a location'
    => an error or Dog at Location :/=
```

The type checker then determines whether that capability exists in the relevant context.

## Structural Satisfaction

Thyrym uses **satisfies** for structural type relationships. `a dog` satisfies `an animal` when the dog provides every capability and guarantee required by the animal type. The dog may provide additional capabilities.

A candidate type satisfies another by satisfying its operator declarations and invariants.

### Operator Satisfaction

A candidate operator satisfies a required operator only if it covers every use promised by the requirement.

#### Pattern Coverage

Every syntactic form accepted by the requirement must be accepted by the candidate. The candidate may accept additional forms. A candidate matching `(House|Home) of ...`, for example, can satisfy a requirement matching `House of ...`.

#### Argument Types

For each argument, the requirement's argument type must satisfy the candidate's argument type. The candidate must accept everything callers are promised they may provide. Arguments are contravariant.

#### Return Types

The candidate's return type must satisfy the requirement's return type. The candidate may return something more specific than promised. Returns are covariant.

> A satisfying operator may accept more than required, but never less; it may return something more specific than promised, but never something outside the promise.

## Declarations And Definitions Inside Types

Declarations inside a type create requirements. A candidate must provide operators satisfying those declarations.

Definitions inside a type do not themselves create additional satisfaction requirements, although a definition may satisfy a declaration. Their possible future role as defaults or reusable capabilities remains open.

## `a thing`

The most general type is:

```thyrym
a thing
```

All ordinary values ultimately satisfy `a thing`. It represents the minimum useful structural guarantee.

## Parameters And Generic Operators

Generic operators use parameters to relate argument and result types:

```thyrym
for all E,
    \'First, a list of Es' + 'Last, a list of Es'
        => a list of Es
        where Length := Length of First + Length of Last
```

A parameter is not intended to appear only in the arguments or only in the return type. Its purpose is specifically to relate the two.

A parameter may specify a minimum bound. Every matched type must satisfy that bound.

When one parameter matches multiple argument types, all matches are combined with `or`:

```text
list of cats + list of cats
-> list of cats

list of cats + list of dogs
-> list of cats or dogs

list of dogs + list of animals
-> list of dogs or animals
```

Structural simplification may reduce a combination. Unrelated values may retain only the guarantees of `a thing`. This is valid; an error occurs only when later code requires a capability the result cannot guarantee.

The parameter itself is a type value available inside the generic implementation. Invariants participate in matching and `or` combinations without a separate generic-invariant mechanism.

## Invariants

Types can carry propositions about their values using `where`:

```thyrym
an integer where it < 0
```

Common cases may have shorter syntax:

```thyrym
an integer < 0
```

A candidate satisfies a required type only when it satisfies both structural and invariant requirements. For example, `an integer where it < -10` satisfies `an integer where it < 0` because the former proposition implies the latter.

### Invariant Reasoning

The invariant solver is not expected to be a complete theorem prover. A proposition may be proven, disproven, or unknown. Ordinary satisfaction requires proof; unknown is insufficient unless the programmer explicitly crosses a trust boundary.

The solver can distinguish interpretations of the same bits, such as signed, unsigned, and floating-point comparisons. Its exact normalization and implication rules remain under development.

Invariant propositions may use exclusive alternatives. If a stopped computation can have either `A and B` or `A and C`, but not both cases, the invariant solver may represent that choice with logical exclusive-or and derive facts common to every case, including `A`.

Logical exclusive-or is not a structural type-composition operator. Structural `or` retains alternatives without asserting that exactly one is possible, while `and` combines required components or capabilities. Exclusion belongs in invariants rather than in a distinct XOR type constructor.

### Trusted Assumptions

Some valid guarantees may be impractical for the solver to prove. The preferred assumption form is:

```thyrym
assuming Invariant
    Next
```

Thyry first attempts to disprove the invariant. A disproven claim fails compilation. A proven claim is accepted normally. Only an unknown claim relies on programmer trust, and the continuation is built in a context containing that assumption.

> Thyrym may allow uncertainty to cross a trust boundary, but not a known falsehood.

Trusted assumptions become part of the trusted computing base.

## `bits`

The only primitive type that directly represents data is:

```thyrym
bits
```

Integers, floating-point values, strings, records, collections, and domain-specific values are built from bits plus type information, invariants, and operators. The same bit representation may support different interpretations and therefore different semantics.

`bits` belongs to the irreducible core because no other data-bearing type can be expressed without it.

## Static Values

`static` is the preferred term for a value whose complete value is available during compilation.

A known equality such as `X = 7` necessarily makes `X` static. Static availability does not have to expose the exact value as part of a public contract; callers may rely on compile-time availability without coupling to a particular constant.

If an operator is referentially transparent and all its arguments are static, its result is static. This provides recursive compile-time evaluation without a separate metaprogramming language. Acts depending on runtime or external state generally stop static propagation.

The exact type syntax for expressing static availability remains open.

## Scoped Uniqueness

Uniqueness is a scoped guarantee arising from meaningful capabilities and invariants rather than a nominal marker. It may be deterministic or generative, structural or relational, or probabilistic.

A structurally unique identifying field can guarantee that filtering by equality returns at most one member. Inserting an arbitrary runtime identifier requires evidence of absence, which a condition can establish for its guarded branch.

Injective operations transport a uniqueness guarantee without changing its source. In particular, an injective transformation preserves a probabilistic collision guarantee but does not turn it into certainty.

The full model is described in [uniqueness](uniqueness.md#scoped-uniqueness).
