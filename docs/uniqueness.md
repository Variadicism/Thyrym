# Scoped Uniqueness

Uniqueness in Thyrym is a family of scoped guarantees, not one universal nominal type. A guarantee must arise from meaningful declarations, capabilities, and invariants. A marker, tag, or stamp with no behavioral consequence does not create a distinct type under this model.

The rationale for this model is recorded in [uniqueness decisions](rationale/uniqueness decisions.md#uniqueness-decisions). Surface syntax and other unsettled details are tracked in [uniqueness questions](unresolved/uniqueness questions.md#uniqueness-questions).

## Sources Of Uniqueness

| Mechanism | Source of guarantee | Example | Failure model |
| --- | --- | --- | --- |
| Deterministic or generative | A construction process is proven not to repeat within its validity scope. | An affine monotonic counter issuing identifiers. | No collision while ownership, overflow, and scope rules hold. |
| Structural or relational | Structure and invariants prevent members of a scope from sharing an identifying relation. | File names within one directory. | Arbitrary input requires an absence check unless non-collision is otherwise proven. |
| Probabilistic | A generator provides enough varying state to keep collision probability below a declared bound. | A UUID-like random identifier. | Collision remains possible, with a scope-dependent statistical bound. |

These mechanisms may compose. A composite identifier can, for example, contain a structurally unique partition and a probabilistic local component. The resulting guarantees retain their original character: an injective transformation of a probabilistic value remains probabilistic.

## Transport Through Injective Operations

An operation transports uniqueness only to the extent that it preserves distinctions relevant to the guarantee. A function `f` is injective over its input when:

```text
f(x) = f(y) -> x = y
```

For a multi-argument operation, full injectivity applies to the complete argument tuple. An operation may instead be injective with respect to one argument while the others are fixed, or preserve only selected information from an argument. Thyrym therefore needs argument-relative and information-relative injectivity rather than only an all-or-nothing function property.

Bit-level examples include:

- `x & 0xF0` preserves the upper nibble and discards distinctions in the lower nibble.
- `x & 0x0F` preserves the lower nibble.
- Separately preserved pieces may be recombined by an injective construction.
- Bitwise OR is not generally injective; OR with a fixed constant preserves only positions where that constant has a zero bit.
- XOR is not injective over two free arguments, but is injective in either argument when the other is fixed.

Linear transformations over `GF(2)` are one mathematical model for bit mappings: full column rank implies injectivity. This is an example of a fact that language-level declarations and invariant reasoning may express, not a requirement for dedicated matrix analysis in the compiler.

A fixed-size cryptographic hash over a larger input domain is not injective. Diffusion, obscurity, and irreversibility are also unrelated to the preservation of uniqueness; an identity or permutation can preserve distinctions exactly.

## Deterministic And Generative Uniqueness

A monotonic identifier source can guarantee non-repetition by construction without examining a collection. An affine counter is the representative model:

- ownership must ensure one coherent sequence of issued values;
- each issued value must differ from every earlier value in the counter's scope;
- overflow must be impossible or handled without repetition;
- concurrent use must preserve one-step, one-identifier semantics;
- the counter must not reset within the same uniqueness scope.

A logical reset creates a new counter associated with a new scope. When the type system can prove these properties, insertion does not need a duplicate lookup solely to establish uniqueness.

## Structural Uniqueness

Structural uniqueness relates a scope, an identifying projection, and the members of that scope. A filename, for example, is not globally unique merely because it is a filename; its name may uniquely identify it within one directory.

### Behavioral Consequence

Uniqueness must have an observable capability. For a collection `C` whose field `K` uniquely identifies members, filtering `C` for a candidate value of `K` returns at most one member:

```text
filter C where member.K = candidate
-> a result where Length <= 1
```

This is stronger than an ordinary filter, whose result is bounded only by the source length. The exact Thyrym declaration syntax is not yet settled.

### Absence Refinement

An arbitrary runtime key cannot be inserted while preserving a structurally unique collection type until non-collision is known. Thyrym's existing condition refinement supplies that evidence. Conceptually:

```thyrym
if Files does not contain NewFile.Name
    Files := Files + NewFile
```

The syntax is illustrative. Inside the guarded branch, the type environment includes the fact that this particular `Files` value lacks this particular name. Combining that fact with the existing collection invariant allows the addition to produce the same declared collection type.

The absence test may execute at runtime. Compilation enforces that its resulting fact, or an equivalent proof, is available before the operation can inhabit the stronger type; the compiler does not assume knowledge of arbitrary runtime collection contents.

No specific lookup implementation is required. A sequence scan, binary search, hash index, tree, or prior static proof may establish absence. The type semantics require the information, not a particular algorithm.

A fallible insertion operator that performs the check internally remains expressible. If it returns a duplicate error, that error must be handled, propagated, retained as data, or explicitly acknowledged as discarded under Thyrym's ordinary error rules. Such an operator is not the foundational mechanism because condition refinement already makes the required evidence available to ordinary invariant-preserving operations.

### Invariant-Preserving Operations

Every operation producing a value intended to retain a uniqueness-constrained collection type must preserve its invariant. This applies to insertion, concatenation, replacement, key mapping, and collection merging. Without proof of properties such as key preservation or cross-collection disjointness, the result cannot retain the stronger type.

Typing also cannot reconstruct information discarded under an earlier application policy. If persisted data omitted a field that a later definition of identity requires, migration is an application-level semantic operation.

## Probabilistic Uniqueness

Probabilistic uniqueness bounds rather than eliminates collision risk. Its guarantee depends on generator properties, the number of identifiers that may coexist, and the accepted collision probability.

### Trusted Generator Metadata

Early Thyrym may treat generator quality as trusted metadata, particularly across a foreign-function boundary. The metadata can conservatively describe:

- total width and data structure;
- invariant or fixed regions that add no distinguishing states;
- random or entropy-bearing regions;
- deterministic regions and the influence determining each region, such as node identity or time;
- effective collision behavior or effective entropy;
- if needed, one conservative distribution factor for the complete value.

Per-bit distribution-quality factors are not used because correlation prevents their naive independent composition.

### Scope And Collision Bound

A deterministic component may partition a population without being entropy. A node identifier that is constant within one process adds no within-process variation but may distinguish identifiers produced on different nodes. Generator metadata identifies the deterministic influence; the use site or environment supplies conservative information about its diversity in the relevant scope.

For approximately independent, uniformly distributed identifiers over `M` effective states and `N` generated values, the birthday approximation is:

```text
P(any collision) ~= N^2 / (2M), when N^2 << M
```

For `k` effectively useful uniform bits and target bound `epsilon`:

```text
k >= ceil(log2(N^2 / (2 * epsilon)))
```

Fixed bits do not increase `M`. A use site must provide enough scope information to bound the population and may provide information about deterministic partitioning. The exact declarations and default collision policy remain unresolved.

### Storage Optimization

When generator structure and use-site bounds are known, an implementation may store only enough distinguishing information to satisfy the required collision bound. Elimination proceeds conservatively:

1. Omit invariant or fixed bits because they distinguish no generated values.
2. Omit deterministic bits that are constant in the relevant scope.
3. Account for scope-dependent partition bits only to the extent that they separate the relevant population.
4. Retain enough random or effectively varying information to meet the collision bound.

The implementation must not claim a stronger probability than the trusted metadata and scope bounds justify. A program can also require the generator's full canonical representation rather than a scope-optimized representation; the exact request syntax remains open.

### Persisted Partial Identifiers

Bits omitted from a persisted optimized identifier are unknown, not zero, and cannot later be reconstructed. If a later deployment expands the scope and would prefer additional original bits, old identifiers remain partial.

Comparison therefore admits three outcomes:

- definitely unequal when a bit known in both values differs;
- definitely equal when the applicable identity contract has sufficient known information to establish equality;
- possibly equal or unknown when all overlapping known bits agree but relevant information is omitted.

A proposed compact textual form is:

```text
<full-width value hex>&<full-width mask hex>
```

The value and mask use the original bit positions. A mask bit of `1` marks represented information; `0` marks omitted information. This permits bit-granular partial values in one string. In-memory position metadata may be shared by a collection or type, while serialized identifiers carry enough metadata for values created under different policies to coexist. The final serialization and equality contracts remain unresolved.
