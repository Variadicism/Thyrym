# Types And Compile-Time Knowledge

Thyrym uses first-class structural types enriched with polymorphic relationships and invariants. The rationale for this model is recorded in [types](../rationale/types.md#type-decisions), and unsettled details are tracked in [types](../unresolved/types.md#type-questions).

## Types Are First-Class Values

Types can be passed as arguments, returned by operators, stored in parameters, and inspected or transformed during compilation.

A type primarily describes operator capabilities and invariants rather than nominal ancestry.

## Structural Identity And Open Membership

Ordinary Thyrym types are extensionally structural. If two type expressions require exactly the same operator capabilities and invariants, they describe the same type regardless of their names, declaring files, or source histories.

Membership is open-world and contextual. A value need not have declared itself as satisfying a type. If the relevant context supplies every required capability and proves every invariant, the value satisfies the type there.

A type may explicitly declare that it satisfies another type. Such a declaration is a promise checked by the compiler rather than a prerequisite for membership: every required capability and invariant still has to be established.

The declaring type is not part of a capability requirement's identity. A deliberately nominal escape hatch may eventually distinguish structurally equivalent types, but no such mechanism is currently part of ordinary type semantics.

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

## Properties And Carried Evidence

A **property** is a dynamically dispatchable operator requirement declared as part of a type and involving its instance placeholder, `one`:

```thyrym
a named := a type with
    \Name of 'one'
        => a string :/=
```

Property-ness belongs to the requirement, not to its implementation. An implementation is an ordinary operator definition:

```thyrym
\Name of 'C, a command' := First Word of C
```

That definition may be called directly in one context and selected as evidence satisfying `named` in another. No separate property-implementation syntax or category is required.

When a value is established as satisfying a type, the compiler selects implementations for the type's property requirements from the context at that satisfaction point. The selected evidence travels with that typed view of the value. Distant code expecting the type can invoke the carried property implementations without having their original declarations in lexical scope.

A property call uses this carried evidence; it does not rediscover the value's hidden subtype and search for fresh capabilities at every call.

The same underlying value may be established as the same structural type in different contexts with different valid property evidence. Re-establishing the type can therefore produce another typed view using newer implementations. Evidence belongs to the particular view of a value as a type, not globally to the raw value.

Whether every property must contain exactly one `one` argument is unresolved.

## Parametric And Existential Abstraction

Generic code may use capabilities exposed by its declared argument type, including dynamically dispatched properties. It does not thereby gain unrelated hidden capabilities of the concrete value. Property dispatch is compatible with parametricity because the property is part of the exposed interface.

Concrete representations may be hidden implicitly behind a structural type. A value typed as `an operator` may have any representation satisfying the operator requirements, and a list of operators may contain heterogeneous implementations without explicit `SomeOperator` wrappers.

The compiler may represent such abstractions with dictionaries, vtables, existential packages, tags, specialization, generated branches, common representations, or another semantics-preserving strategy. Source semantics do not select a representation.

Recovering relationships between independently hidden values is not settled. In particular, exposed property evidence does not grant unrestricted runtime reflection over every hidden structural capability.

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

When one parameter matches several actual types, Thyrym retains their exact structural `or`. Repeated identical bindings collapse only because `A or A` is `A`:

```text
GLSL float + GLSL float
-> GLSL float

IEEE float + GLSL float
-> IEEE float or GLSL float

cat + dog
-> cat or dog
```

The compiler does not automatically widen `IEEE float or GLSL float` to `float`, or `cat or dog` to `pet`, even when each alternative satisfies that broader type. The user may explicitly accept or infer the broader type when the extra specificity is no longer needed.

For example, a user-directed result annotation may accept an exact parameter result as its broader capability type:

```thyrym
Result, a float := SpecializedLeft + SpecializedRight
```

An `or` value guarantees every capability shared by its alternatives, but retaining the exact alternatives preserves more information than replacing them with only that common capability set.

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

This differs from returning `F`. `F` is the exact `or` of the types bound by all its occurrences; `same as Augend` follows only the complete specialized type of `Augend` directionally.

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

Returning `same as State` preserves the state argument's specialization. Returning `a distinct state of A` explicitly requests an independent state specialization. Returning a shared parameter such as `F` uses the exact `or` accumulated from all occurrences of `F`.

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

## `or` Types

`A or B` is inclusive: a value may satisfy `A`, `B`, or both. Structural typing generally cannot express XOR by asserting the absence of extra capabilities. Logical exclusion remains available in invariants where a proposition can meaningfully be negated.

Knowing only that `X` has type `A or B` does not currently imply settled runtime tagged-union discrimination. Whether code may discover which alternatives the current value satisfies is deferred alongside runtime structural reflection.

## Implicit Conversions

An implicit conversion is permitted only when it is unambiguous. If several conversions could make a call valid, the compiler does not guess; the programmer must select one explicitly.

Every implicitly available conversion must also have an explicit form. This prevents hidden compiler preference and circular inference while preserving convenient conversions whose meaning is unique.

## Minimal Inference

Thyrym resolves declared constraints rather than broadly inventing likely intended types:

- operator argument types are explicit;
- an implementation satisfying a property may inherit argument types from that property requirement;
- bodyless declarations state an explicit return type;
- definitions may state a return type, otherwise they receive the exact type of their body expression.

Expected return types can constrain operator resolution, but they do not cause speculative widening of inferred values.

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

Compiler versions may add proof strategies, stronger solvers, and configurable proof-resource limits without changing these semantic outcomes.

The solver can distinguish interpretations of the same bits, such as signed, unsigned, and floating-point comparisons. Its exact normalization and implication rules remain under development.

Invariant propositions may use exclusive alternatives. If a stopped computation can have either `A and B` or `A and C`, but not both cases, the invariant solver may represent that choice with logical exclusive-or and derive facts common to every case, including `A`.

Logical exclusive-or is not a structural type-composition operator. Structural `or` retains alternatives without asserting that exactly one is possible, while `and` combines required components or capabilities. Exclusion belongs in invariants rather than in a distinct XOR type constructor.

### Inhabitation

A type whose invariants are proven contradictory is invalid. For example, `an integer where it > 10 and it < 0` is rejected when the solver proves the contradiction.

Unknown inhabitation does not make a type valid automatically. It follows the ordinary proof, explicit checked-refinement, and assumption rules. If an intentionally empty type becomes useful, it should have an explicit representation rather than disguise itself as contradictory invariants; that representation is not designed.

### Explicit Checked Refinement

An unknown invariant may be tested through an explicit runtime check whose success and failure are represented in typed control flow or a typed result. Only the successful outcome refines the value with the invariant.

The compiler does not silently turn every unknown proof obligation into a potentially failing runtime assertion. Explicit checked refinement preserves the rule that an unproven invariant does not satisfy a requirement and preserves Thyrym's prohibition on hidden language-level runtime errors. Exact checking syntax remains open.

### Trusted Assumptions

Some valid guarantees may be impractical for the solver to prove. The preferred assumption form is:

```thyrym
assuming Invariant
    Next
```

Thyry first attempts to disprove the invariant. A disproven claim fails compilation. A proven claim is accepted normally. Only an unknown claim relies on programmer trust, and the continuation is built in a context containing that assumption.

An assumption is invalid if any case it admits is proven false, even when other cases remain unknown. Assumptions complete missing proof knowledge; they are not casts that discard known alternatives. A value known only as `A or B` cannot be assumed to be `A` when admitted `B` values are known not to satisfy `A`.

> Thyrym may allow uncertainty to cross a trust boundary, but not a known falsehood.

Trusted assumptions become part of the trusted computing base.

## Laws

An invariant constrains a particular value or result and conventionally refers to `it`. A **law** instead supplies universally quantified knowledge to the solver.

Conceptually, a user-defined law may take a form such as:

```thyrym
assuming for all A, B, and C,
    ...
```

The exact syntax may evolve; no separate `law` keyword is currently required. Because general semantic laws are not normally derivable from implementation syntax, they are assumptions subject to the same restriction as other assumptions: any proven admitted counterexample rejects the law.

Laws are solver knowledge, not obligations to carry a theorem prover or exhaustive proof search into ordinary executables.

Potentially ongoing computation is modeled through pursuits rather than as a special invariant mechanism, as described in [pursuits](pursuits.md#convergence-and-divergence).

## Declaration Proof Obligations

Operator and property declarations may promise strong return invariants. Every definition satisfying such a declaration must prove that all values it can produce meet those invariants.

The implementation may rely on invariants and laws promised by lower-level operators. Semantic guarantees therefore compose through the solver rather than through a separate refinement-compatibility mechanism.

## Parameterized Capabilities And Heterogeneous Values

Parameterized types remain ordinary type-producing operators checked through capabilities and invariants. `a collection of strings` is satisfied only when the collection's relevant operations have the relationships required for strings.

Relationships between values can likewise be structural capabilities such as `equatable to T` or `comparable to T`; they do not require recovering a hidden nominal identity. No comparison-specific variance system is needed beyond ordinary operator substitutability.

A heterogeneous collection does not automatically guarantee every pairwise relationship among its elements. A list of operators is sortable only when its element type exposes comparison for every admitted pair or the caller supplies an explicit common comparison mechanism. Incomparable values must not silently be treated as equal.

## `bits`

The only primitive type that directly represents data is:

```thyrym
bits
```

Integers, floating-point values, strings, records, collections, and domain-specific values are built from bits plus type information, invariants, and operators. The same bit representation may support different interpretations and therefore different semantics.

`bits` belongs to the irreducible core because no other data-bearing type can be expressed without it.

The compiler may understand a small additional set of native substrate types needed to implement the language, but privileged types should remain minimal. Ordinary user-defined types participate through the same structural capability and invariant rules wherever possible.

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
