# Modules And Imports

The import system treats a source file approximately as an exportable operator context. Its rationale is recorded in [modules](../rationale/modules.md#module-decisions), and unsettled details are tracked in [modules](../unresolved/modules.md#module-questions).

The model remains provisional. Only the context established by an imported file itself is imported; operators merely imported by that file do not leak transitively.

## Whole-File Imports

The intended syntax is approximately:

```thyrym
using "file.thy"
```

## Selected Operators

Individual operators may be imported:

```thyrym
using Format from "logging.thy"
```

Multiple operators use ordinary Thyrym list syntax:

```thyrym
using Format and Level from "logging.thy"
```

## Paths Are Values

Import paths use ordinary string expressions rather than a special filename grammar. A path must be static and satisfy the required path constraints.

```thyrym
Directory := "lib/"
using Format from Directory + "logging.thy"
```

This is valid when the resulting path is static and valid.

## Local Imports

Imports may be scoped locally. Conceptually, `using` loads an operator context, consumes a following pre-op, and builds that pre-op using the augmented context.

## Import Order

Import order participates in normal operator-stack priority. Later imported applicable operators have higher priority.

When an explicitly imported operator is obscured by a later import, a compiler warning would be useful. A program can resolve conflicts by reordering imports, selecting only required operators, or reducing an import to a narrower scope.

## Visibility

The exact export and private model is unresolved. Operators needed only in a local scope should preferably be defined in that scope rather than hidden through a modifier.

The difficult case is a helper shared by several top-level exported operators but not itself intended for export.

## Compilation Units

Separate compilation, compiled-module metadata, and the precise representation of exportable operator contexts remain future work.
