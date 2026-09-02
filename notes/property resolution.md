Let's stick with the "named" example because it's simple.

### Thyrym code

`named.thm`:
```
\named 'Type, a type' := a Type with \name of 'one' :: a string
```

`command.thm`:
```
using argument from ./arguments.thm:

\command := a thing with
	First Word :: a string
	Arguments :: a list of arguments
	\run 'one' with 'Tokens, a list of strings' :: an act making an error or nil

\Name of 'Command, a command' := First Word of Command
```

`mkdir.thm`:
```
using a file path argument from ./arguments.thm:
using ./command.thm:

\mkdir := a command with
	First Word := "mkdir"
	Arguments := a file path argument
	\run 'one' with 'Tokens' := create a directory at index 0 of Arguments
```

`custom nullary.thm`:
```
using ./commands.thm:

# This "for any" is really just to factor out the long type for readability.
for any runner, a list of strings -> an act making an error or nil,
\custom nullary command 'First Word, a name' doing 'Runner, a runner' := a command with
	Name := First Word + " (custom)"
	Arguments := nil
	First Word
	\run 'one' with 'Tokens' := Runner given Tokens
```

`run.thm`:
```
using ./named.thm:
using ./commands.thm:
using ./mkdir.thm:
using ./custom nullary.thm:
# CODA

Commands :: a list of named commands unique by First Word :=
	mkdir
	a custom nullary command named "streaming-sinks" doing [...]  # placeholder for implementation
	a custom nullary command named "remove-pa-null-sinks" doing [...]  # placeholder for implementation

\run 'Name, a string' with 'Tokens, a list of strings' :=
	for each Command in Commands,
		if First Word of Command = Name,
			run Command with Tokens

\print help page :=
	for each Command in Commands,
		print Name of Command + ": " + First Word of Command + " " + Argument of Command verbosely
```

## Context-first

- Calling `print help page` will print three lines starting with "mkdir: ", "streaming-sinks (custom): ", and "remove-pa-null-sinks (custom)".
- `First Word` and `Name` are the same by default (established in `commands.thm`), but semantically a bit different: `Name` is meant to be human-readable while `First Word` is the identification keyord for the command.
- `commands.thm` defined a Name for all commands, ensuring that `mkdir` and outputs of `a custom nullary command...` are both treated as `named` and allowed in the `Commands` list of named commands.
	- But, `\custom nullary command...` provides its own implementation of `Name` as a property.

When the compiler checks that `mkdir` is `a named command`,...
	- It sees that there is no implementation of `Name of 'mkdir'` in `run.thm`.
	- It sees none in `custom nullary.thm`.
	- It sees none in `mkdir.thm`.
	- It finds the implementation from `commands.thm` and verifies that `mkdir` is indeed `named`.
At runtime, `print help page` resolves `Name of Command` when `Command` is `mkdir` to the definition from `command.thm`.

When the compiler checks that `a custom nullary command...` is `a named command`,...
	- It sees that there is no implementation of `Name of 'mkdir'` in `run.thm`.
	- It finds the implementation from `custom nullary.thm` and verifies that the custom commands are indeed `named`.
At runtime, `print help page` resolves `Name of Command` when `Command` is `a custom nullary command...` to the definition from `custom nullary.thm`.

### A potentially confusing case

This is where it might seem a bit strange and I might be open to changing this to a different (prop-first) strategy.

If you moved the import of `commands.thm` down to the `# CODA` comment line, then its definition of `Name of 'Command'` would be the most recent.

Therefore, `print help page` would actually print lines starting with "mkdir: ", "streaming-sinks: " and "remove-pa-null-sinks: ", ignoring the `Name of 'one'` implementation of the custom nullary commands!

But, this would only apply within `run.thm` (after the import of `commands.thm`). If some other file import them in the order of the original example, then the implementation available from `custom commands.thm` would be used.

### Intuitive or not?

This may seem unintuitive, but consider the case where the imports remain the way they are, but at the `# CODA`, we insert `using ./named by first.thm:` which contains the following:
```
\Name of 'Command, a command' := First Word of Command
```

Now that file's explicit purpose is to ensure that naming happens in a certain way in this file and it accomplishes that by appearing later in the context stack.

### Resolution: compiler warning

Make the compiler warn about shadowing from imports.

The warning will disappear if the import of that operator is made explicit like this:
```
using \Name of 'Command' :: a string from ./named by first.thm:
```