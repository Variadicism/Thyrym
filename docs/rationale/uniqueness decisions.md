# Uniqueness Decisions

This document records the reasoning behind the scoped [uniqueness](../uniqueness.md#scoped-uniqueness) design without extending its guarantees.

## Behavioral Guarantees Instead Of Markers

A nominal `unique` marker, stamp, or tag was rejected because Thyrym types differ through meaningful declarations, capabilities, and invariants. Scoped uniqueness instead strengthens observable behavior, such as proving that filtering a collection by its identifying field yields at most one member.

## Existing Refinement Instead Of Special Construction

Restricted constructors and proof tokens are unnecessary for ordinary structural uniqueness. A condition that establishes absence already refines the guarded type environment, allowing the normal collection operation to prove that it preserves the invariant.

A mandatory fallible insertion protocol was also rejected as the foundational model. Guard-first refinement exposes the absence fact to ordinary operations and can support more than insertion without introducing a special result protocol. APIs may still return result types when useful, but a duplicate error must be handled or propagated under Thyrym's ordinary error rules.

This design distinguishes compile-time enforcement from runtime knowledge. For arbitrary input, the absence query may run at runtime; compilation checks that the established fact governs the invariant-preserving path.

## Generative Proofs Avoid Redundant Lookups

A provably non-repeating affine source already establishes non-collision in its scope. Requiring a collection lookup in addition would add work without adding evidence, provided ownership, overflow, concurrency, and scope reuse are all controlled.

## Injectivity Preserves Rather Than Strengthens

Injectivity is the relevant transport property because it preserves distinctions. It does not require diffusion, obscurity, or irreversibility, and it does not upgrade a probabilistic guarantee to certainty.

Fixed-size cryptographic hashes over larger domains were rejected as examples of injectivity because they necessarily collide. General claims that OR or two-argument XOR are injective were also rejected; injectivity must be stated relative to the argument and information actually preserved.

## Scope-Aware Probabilistic Metadata

Deterministic generator fields are not called entropy merely because they sometimes separate values. Their contribution depends on scope: a node field constant within one process does not add local variation but can partition a distributed population.

Fuzzy per-bit distribution factors were rejected because correlation makes their composition misleading. Explicit fixed, random, and deterministically influenced regions, together with conservative whole-value collision behavior, provide a sounder basis.

Generator quality may initially be trusted because Thyrym cannot prove arbitrary random or foreign implementations statistically sound. This follows the language's existing trusted-boundary model while allowing downstream preservation and optimization to be checked precisely.

## Partial Storage Is Explicitly Lossy

Scope-bounded probabilistic identifiers may omit information that does not contribute to the required collision bound, reducing storage. This optimization is not represented as reversible: omitted persisted bits are unknown and unavailable to later policies.

Explicit masks and partial-information comparison prevent omitted bits from being treated as zero or silently reconstructed. A canonical full representation remains available for uses that require the generator's complete value.
