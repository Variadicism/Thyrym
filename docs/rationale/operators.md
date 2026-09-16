# Operator Decisions

This document records the reasoning behind Thyrym's [operators](../design/operators.md#operators-and-grammar) without extending their guarantees.

## Operators As The Callable Abstraction

Using operators for all callable behavior avoids a separate function mechanism and lets ordinary computation, declarations, syntax extension, binding, and contextual constructs share one model. English-inspired grammar makes that flexibility readable without claiming source is literal English.

Pure operators generally use descriptive wording because they identify immutable results. Command-like wording is reserved primarily for acts, where it naturally describes an effect.

## Newest Applicable Resolution

Choosing the newest applicable operator combines lexical shadowing with overloading. A newer syntactic match does not hide an older operator when its arguments, result, or invariants do not satisfy the use, so local specialization can coexist with broader definitions.

## Pre-Ops And Compile-Time Grammar

Pre-ops delay parsing and type checking until a construct has introduced the context required by its body. Branch refinements, loop variables, bindings, and scoped domain-specific syntax can therefore use one mechanism.

Generated grammar is restricted to compile-time-known information so every pre-op can be resolved before code emission. Runtime input cannot alter grammar, preserving complete compilation and static checking.

## Conventional And Deviant Syntax

The proposed `deviant` marker preserves unrestricted syntax while making a readable common dialect recognizable. Conventional forms could also support forward use because the compiler can recognize their shape before seeing a definition; unusual grammar continues to require an explicit declaration.
