# Operator Decisions

This document records the reasoning behind Thyrym's [operators](../design/operators.md#operators-and-grammar) without extending their guarantees.

## Operators As The Callable Abstraction

Using operators for all callable behavior avoids a separate function mechanism and lets ordinary computation, declarations, syntax extension, binding, and contextual constructs share one model. English-inspired grammar makes that flexibility readable without claiming source is literal English.

Pure operators generally use descriptive wording because they identify immutable results. Command-like wording is reserved primarily for acts, where it naturally describes an effect.

## Newest Applicable Resolution

Choosing the newest applicable operator combines lexical shadowing with overloading. A newer syntactic match does not hide an older operator when its arguments, result, or invariants do not satisfy the use, so local specialization can coexist with broader definitions.

“Most specific overload wins” was rejected as the default because contextual precedence is what permits deliberate local replacement. Specificity matters only through satisfying the complete call.

Return requirements participate during resolution because selecting first and checking later could reject a program even when an older candidate has the required result. With no expected result, preserving the selected definition's exact return type avoids guessing user intent.

## Static And Property Competition

Static and property declarations share one context stack because placing properties in a privileged namespace would let them ignore a newer local operator explicitly introduced by the programmer. Their semantic distinction begins after resolution: static calls bind directly, while property calls dispatch through evidence selected when type satisfaction was established.

Property implementations remain ordinary operators so one definition can be called statically, satisfy a property in another context, and later be dispatched through that property without duplicate declaration mechanisms.

## Backtracking

A locally plausible interpretation may conflict with constraints discovered later. Allowing resolution to backtrack preserves the rule that the newest declaration wins among interpretations that make the program valid, rather than committing prematurely based on incomplete context.

Looking ahead does not make later declarations retroactive. Lexical availability and the amount of source needed to validate a choice are separate concerns.

## Shadowing Diagnostics

Shadowing is semantically useful but easy to do accidentally through imports or a near-matching signature. Warnings and resolution explanations can expose those mistakes without weakening contextual override semantics.

## Pre-Ops And Compile-Time Grammar

Pre-ops delay parsing and type checking until a construct has introduced the context required by its body. Branch refinements, loop variables, bindings, and scoped domain-specific syntax can therefore use one mechanism.

Generated grammar is restricted to compile-time-known information so every pre-op can be resolved before code emission. Runtime input cannot alter grammar, preserving complete compilation and static checking.

## Conventional And Deviant Syntax

The proposed `deviant` marker preserves unrestricted syntax while making a readable common dialect recognizable. Conventional forms could also support forward use because the compiler can recognize their shape before seeing a definition; unusual grammar continues to require an explicit declaration.
