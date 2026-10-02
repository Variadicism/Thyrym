# Operator Questions

This document tracks questions left unresolved by the [operators](../design/operators.md#operators-and-grammar) design. Statements here are not guarantees until their answers are incorporated into the design.

## Conventional Operator Forms

Which grammatical shapes should count as conventional, non-`deviant` forms?

The forms must be constrained enough for reliable forward recognition while covering ordinary syntax. The recognition algorithm also remains to be designed.

## Completing Forward Declarations

How should an implementation attach to an existing operator declaration?

The mechanism should identify the declaration without requiring a complicated pattern to be repeated.

## Warning Suppression And Resolution Diagnostics

What syntax marks intentional shadowing or suppresses its warning outside an explicit selective import?

What information should an `explain resolution` diagnostic provide about rejected candidates, return constraints, invariant failures, and backtracking?

These are tooling and surface-language questions; newest-valid contextual resolution is already the semantic rule.

## Static Operator Terminology

Should **static op** become the final term for an ordinary context-resolved operator contrasted with a dynamically dispatched property requirement?

The semantic distinction is established, but the label remains provisional.
