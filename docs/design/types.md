# Types And Compile-Time Knowledge

Thyrym uses first-class structural types enriched with polymorphic relationships and invariants. The rationale for this model is recorded in [types](../rationale/types.md#type-decisions), and unsettled details are tracked in [types](../unresolved/types.md#type-questions).

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

Repeated broad structural type phrases do not implicitly denote the same specialization. In:

```thyrym
\'Augend, a float' + 'Addend, a float'
    => a float
```

each `a float` may independently be satisfied by a different specialized type. The declaration alone does not say that both arguments share a specialization or that the result preserves either argument's specialization.

Generic operators use parameters to state shared relationships explicitly. The current introductory form is `for any`:

```thyrym
for any F, a float,
    \'Augend, F' + 'Addend, F'
        => F
```

`F` is a **parameter** with `a float` as its minimum bound. Every occurrence must satisfy that bound. A parameter is a relationship carrier rather than a requirement that every argument have one exact nominal or runtime type.

When one parameter matches several actual types, Thyrym combines them using its existing structural `or` behavior. The resulting guaranteed capability set is their common intersection. Structural simplification may retain a more specific common type:

```text
GLSL float + GLSL float
-> GLSL float

IEEE float + GLSL float
-> float

cat + dog
-> cat or dog
```

The final alternative guarantees only capabilities shared by cats and dogs. Unrelated values may therefore retain only the guarantees of `a thing`; an error occurs only when later code requires a capability the combination cannot guarantee.

The parameter itself is a type value available inside the generic implementation. Invariants participate in matching and `or` combinations without a separate generic-invariant mechanism.

A parameter may be intentionally unconstrained when the algorithm does not require a narrower capability:

```thyrym
for any A and B,
    \'Function, A -> B' mapped over 'Values, a list of As'
        => same as Values but Element := B
```

Authors should state only restrictions required by the implementation. Descriptive structures such as `a list of As` remain inline instead of being replaced with opaque parameters unless the identity of the list family itself matters.

The punctuation for complex `for any` lists remains provisional. Current examples use `Name, a type`, may separate complex entries with semicolons, and reserve `=>` for the boundary before the return type.

### Constraints

Parameters and argument types use the ordinary invariant system rather than a separate generic-constraint language:

```thyrym
for any I, an integer,
    \'Numerator, I' / 'Denominator, I /= 0'
        => I
```

The longer `I where it /= 0` form remains available when it reads more clearly or the proposition is more involved.

## Type Relations

A **relation** states how a type depends on something already in scope. Relations complement parameters: parameters express shared or symmetric inference, while relations express directional preservation or intentional independence.

### `same as`

`same as Argument` preserves the type of a particular argument:

```thyrym
for any F, a float,
    \'Augend, F' * 'Addend, F'
        => same as Augend
```

This differs from returning `F`. `F` is the common type inferred from all its occurrences; `same as Augend` follows the complete specialized type of `Augend` directionally.

A working explicit-intersection form is:

```thyrym
same as Augend + Addend
```

It denotes the common structural capabilities of the referenced argument types. Its exact integration with the solver remains under design.

### `same as ... but ...`

`but` can modify one exposed role while preserving the rest of an argument's structural type:

```thyrym
for any A and B,
    \'Function, A -> B' mapped over 'Values, a list of As'
        => same as Values but Element := B
```

The result keeps the specific collection structure supplied as `Values` and changes only its element role. No separate parameter for the list family is required.

This directional form also avoids false equality between arguments:

```thyrym
for any A and B,
    \'Left, a list of As' combined with 'Right, a list of Bs'
        => same as Left but Element := (A, B)
```

`Left` may be an array and `Right` a linked list when the implementation only preserves the representation of `Left`.

### `a distinct ...`

> **Working syntax:** The independence concept is established, but its exact grammar and unification semantics remain open.

`a distinct Type` states that a result intentionally chooses an independent specialization satisfying `Type` instead of preserving an argument's specialization:

```thyrym
a distinct state of A
```

**Distinct** is preferred to **new**, which could misleadingly suggest allocation or object construction.

### Parameters, Relations, And Quantifiables

The working terminology is:

- **parameter**: a universally quantified type variable introduced by `for any`;
- **relation**: an explicit dependency or independence form such as `same as` or `a distinct`;
- **quantifiable**: a provisional umbrella term for parameters and relations.

## Return-Type Relationships

> **Emerging rule:** The exact recursive enforcement and its interaction with monads are unresolved.

A return type should not silently introduce fresh polymorphic structure by restating a broad type and losing argument specialization. Its variable parts should instead be derived through parameters or explicit relations.

Returning `same as State` preserves the state argument's specialization. Returning `a distinct state of A` explicitly requests an independent state specialization. Returning a shared parameter such as `F` uses the common type inferred from all occurrences of `F`.

Concrete wrappers require more work. A form such as `an act making same as State` is attractive, but higher-order mapping must also preserve the exact act, task, or other monad specialization returned by the mapped operator. Nested relation projection and monadic propagation are not yet settled.

## Type-Producing Operators And Families

A generic type constructor is an ordinary operator whose result is a type, not a separate compiler category. Parameters can distinguish several independently selected types satisfying one family from repeated applications of the same type-producing operator.

When family identity matters, a shared family parameter may be appropriate. It should not be introduced merely to replace readable phrases such as `a list of As` with letters.

For example, this result follows the vector argument without requiring the matrix and vector to use one representation family:

```thyrym
for any D and N,
    \'Matrix, a DxD matrix of Ns'
        * 'Vector, a vector of D Ns'
        => same as Vector
```

If the operation genuinely requires one tensor family for both operands and the result, it can instead introduce a shared family parameter:

```thyrym
for any D; N; and T, a tensor,
    \'Matrix, T of [D, D] Ns'
        * 'Vector, T of [D] Ns'
        => T of [D] Ns
```

These family-application forms remain advanced and partly provisional. Their semantic distinction is important: independently bounded type parameters may select unrelated implementations, while repeated application of one `T` preserves family identity.

Thyrym's roles are named rather than positionally significant, so positional partial application is not the foundation for type families. The placeholder `it` may identify one missing role in a partial type application. Applying a family with several open roles likely requires explicit role names; that syntax remains exploratory.

## Capability-Portable Polymorphism

Types describe semantic capabilities rather than execution environments. Numeric distinctions should therefore express properties such as general float behavior, 16-bit behavior, or IEEE-specific operations rather than nominal CPU and GPU categories.

A GLSL expression type can satisfy the general float capability and provide `+` or `*` implementations that construct shader expressions. The same source operator applied to ordinary floats performs arithmetic. A pure generic expression such as `(A + B) * C` can then be reused for CPU evaluation or shader construction, provided its parameters and return relations preserve the supplied specialization.

An operator using IEEE-specific representation or bit operations must require the IEEE capability and will not automatically accept a GLSL expression that provides only general float behavior.

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
