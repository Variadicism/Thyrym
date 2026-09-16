# Operator Questions

This document tracks questions left unresolved by the [operators](../design/operators.md#operators-and-grammar) design. Statements here are not guarantees until their answers are incorporated into the design.

## Conventional Operator Forms

Which grammatical shapes should count as conventional, non-`deviant` forms?

The forms must be constrained enough for reliable forward recognition while covering ordinary syntax. The recognition algorithm also remains to be designed.

## Completing Forward Declarations

How should an implementation attach to an existing operator declaration?

The mechanism should identify the declaration without requiring a complicated pattern to be repeated.
