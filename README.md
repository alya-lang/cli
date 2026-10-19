# cli

[![CI](https://github.com/alya-lang/cli/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/cli/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/cli?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fcli%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fcli%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

Modern command-line toolkit for Alya with typed flags, options, subcommand routing, generated help screens, and structured parse errors.

---

## 🌟 Features

- ⚡ **Native & Zero-Dependency**: Pure Alya implementation, no runtime dependencies, compiled by the Alya toolchain
- 🧩 **Layered Architecture**: Public facade (`src/lib.alya`), data models (`src/types.alya`), and focused engines (`src/core/`)
- 🎛️ **Complete Option Grammar**: Long (`--output dist`), inline (`--output=dist`), short (`-o dist`), attached (`-oDist`), clustered (`-abc`), negated (`--no-color`), repeatable (`-I a -I b`), and the `--` terminator
- 🌳 **Hierarchical Subcommands**: Nested command trees, per-command options and arguments, and short aliases (`build` ⇄ `b`)
- 🛡️ **Validation & Diagnostics**: Required options, choice lists, required arguments, unknown options/commands — collected as structured `CliError` values instead of aborting
- 🧾 **Generated Usage Screens**: Column-aligned help screens for the application and for every command, with defaults, choices, and environment hints
- 🌍 **Environment Fallback**: Options can read their value from an environment variable when omitted on the command line
- 🔒 **Public/Private Visibility**: `pub` marks the public surface; the shared internals in `src/core/internal.alya` stay private to the package
- 🧪 **Dual API Style**: A fluent builder (`app.flag(...)`) and a functional API (`cli::add_flag(app, ...)`) over the same engines
- 🚩 **Feature-Gated Slices**: `help` (usage screens), `strict` (throwing/recovering parses), and `env` (environment fallback) can be enabled independently
- 🧬 **Modern Alya**: Structural interfaces, `enum` vocabularies, nullable types, `try`/`catch` recovery, and typed accessors (`get_int`, `get_bool`)

---

## 📁 Project Architecture

```
cli/
├── .alyafmt               # Formatter configuration
├── .alyalint              # Linter configuration (rules, exclusions, severity overrides)
├── .alyatest              # Test runner configuration
├── .editorconfig         # Uniform formatting rules across IDEs and editors
├── .gitattributes         # Line ending and diff rules
├── .gitignore             # Ecosystem standard ignore filters
├── alya.toml              # Package manifest with dependencies, [features] and metadata
├── LICENSE                # MIT license
├── README.md              # This document
├── .github/               # CI, release, code scanning, and issue templates
├── .vscode/               # VS Code workspace settings, DAP launch configurations & tasks
├── src/
│   ├── lib.alya           # Public API facade (thin wrappers delegating to core/* engines)
│   ├── types.alya         # Data models, pub enums, pub structs, factories & error helpers
│   └── core/
│       ├── spec.alya      # Specification builders: applications, commands, options, arguments
│       ├── parser.alya    # Parsing engine: token classification, binding, validation
│       ├── formatter.alya # Usage screen, version banner, and diagnostic rendering (`help`)
│       ├── errors.alya    # Strict parsing slice: throwing and recovering entry points (`strict`)
│       └── internal.alya  # Private string and model helpers shared inside the package
├── tests/
│   ├── test_basic.alya    # Application setup, flags, options, typed accessors
│   ├── test_flags.alya    # Clustered, negated, attached, and repeatable options
│   ├── test_commands.alya # Subcommands, aliases, nesting, command-scoped options
│   ├── test_validation.alya # Required options, choice lists, unknown input
│   ├── test_formatter.alya # Usage screens and diagnostic reports (`help` slice)
│   └── test_strict.alya   # Strict parsing and error recovery (`strict` slice)
├── examples/
│   └── demo.alya          # Comprehensive runnable walkthrough of the package
└── benches/
    └── bench_basic.alya   # Micro-benchmarks for the hot paths
```

> [!NOTE]
> **Visibility & Modularity:** Symbols annotated with `pub` are exported to external consumers. `src/core/internal.alya` declares no `pub` symbol on purpose: an Alya module without public items shares its whole surface with sibling modules, so the shared helpers stay reachable inside the package while remaining invisible to package consumers.

---

## 📦 Installation

Add `cli` to the `[dependencies]` section in your `alya.toml`:

```toml
[dependencies]
cli = { git = "https://github.com/alya-lang/cli", branch = "main" }
```

Or install it directly using the Alya package CLI:

```bash
alya add cli --git https://github.com/alya-lang/cli --branch main
alya install
```

### Package Features

| Feature | Default | Description |
|:---|:---:|:---|
| `help` | ✅ | Usage screen, version banner, and diagnostic report rendering (`format_help`, `format_command_help`, `format_version`, `format_errors`, `print_help`, `print_command_help`, `print_version`). |
| `strict` | ✅ | Throwing and recovering parse entry points (`parse_strict`, `try_parse`) on top of the lenient engine. |
| `env` | ✅ | Environment variable fallback for options registered with `env_option`. |

```bash
# Full build (default)
alya install
alya test

# Slim build (parsing only)
alya install --no-default-features
alya test --no-default-features
```

---

## 🚀 Quick Start

```alya
import "cli" as cli

function main()
    # 1. Describe the application
    let app = cli::new_app("forge", "Alya build and test tool", "1.0.0")

    # 2. Register the global options
    app.flag("-v, --verbose", "Enable verbose logs")
    app.option("-o, --output", "dist", "Output directory")

    # 3. Register a subcommand with its own options and arguments
    let build = cli::new_command("build", "Compile the target package", ["b"])
    build.flag("-r, --release", "Build optimized artifacts")
    build.argument("entrypoint", "Source file to compile", false, "src/main.alya")
    app.command(build)

    # 4. Parse the arguments and read the resolved values
    let ctx = app.parse(["build", "-v", "--release", "server.alya"])
    if ctx.has_errors()
        say cli::format_errors(ctx.get_errors())
        exit(1)
    end

    if ctx.has_command("build")
        say "Release: " + str(ctx.get_flag("release"))
        say "Entry   : " + ctx.get_arg_named("entrypoint", "src/main.alya")
    end
end

main()
```

---

## 📖 API Reference

### Application & Command Construction

| Symbol | Visibility | Description |
|---|---|---|
| `new_app(name, description, version)` | `pub function` | Creates an application specification. |
| `new_command(name, description, aliases)` | `pub function` | Creates a command specification. |
| `set_author(app, author)` | `pub function` | Sets the author string of an application. |
| `set_version(app, version)` | `pub function` | Sets the version string of an application. |

### Option & Argument Registration

| Symbol | Visibility | Description |
|---|---|---|
| `add_flag(app, spec, description)` | `pub function` | Registers a boolean flag (`-v, --verbose`). |
| `add_option(app, spec, default, description)` | `pub function` | Registers a single-value option (`-o, --output`). |
| `add_multi_option(app, spec, description)` | `pub function` | Registers a repeatable option (`-I, --include`). |
| `add_choice_option(app, spec, default, choices, description)` | `pub function` | Registers an option restricted to a choice list. |
| `add_required_option(app, spec, description)` | `pub function` | Registers a mandatory option. |
| `add_env_option(app, spec, env_var, default, description)` | `pub function` | Registers an option falling back to an environment variable. |
| `add_argument(app, name, description, required, default)` | `pub function` | Registers a positional argument. |
| `add_variadic_argument(app, name, description)` | `pub function` | Registers an argument capturing the remaining values. |
| `add_command(app, command)` | `pub function` | Registers a top-level command. |
| `command_add_flag(cmd, spec, description)` | `pub function` | Registers a command-scoped flag. |
| `command_add_option(cmd, spec, default, description)` | `pub function` | Registers a command-scoped option. |
| `command_add_multi_option(cmd, spec, description)` | `pub function` | Registers a command-scoped repeatable option. |
| `command_add_choice_option(cmd, spec, default, choices, description)` | `pub function` | Registers a command-scoped choice option. |
| `command_add_required_option(cmd, spec, description)` | `pub function` | Registers a command-scoped mandatory option. |
| `command_add_env_option(cmd, spec, env_var, default, description)` | `pub function` | Registers a command-scoped environment option. |
| `command_add_argument(cmd, name, description, required, default)` | `pub function` | Registers a command-scoped positional argument. |
| `command_add_variadic_argument(cmd, name, description)` | `pub function` | Registers a command-scoped variadic argument. |
| `command_add_subcommand(parent, child)` | `pub function` | Nests a command under another command. |

### Parsing

| Symbol | Visibility | Description |
|---|---|---|
| `parse(app, raw_args)` | `pub function` | Parses an argument array into a `CliContext`. |
| `parse_args(app)` | `pub function` | Parses the process arguments into a `CliContext`. |
| `parse_strict(app, raw_args)` | `pub function` (`strict` feature, default-on) | Parses and throws a structured `CliError` on the first diagnostic. |
| `try_parse(app, raw_args)` | `pub function` (`strict` feature, default-on) | Parses and converts a failure into a `CliResult`. |

### Context Queries

| Symbol | Visibility | Description |
|---|---|---|
| `get_flag(ctx, name)` | `pub function` | Returns the state of a parsed flag. |
| `get_option(ctx, name, default)` | `pub function` | Returns the value of a parsed option. |
| `get_int(ctx, name, default)` | `pub function` | Returns the value of a parsed option converted to an integer. |
| `get_bool(ctx, name, default)` | `pub function` | Returns the boolean value of a parsed option. |
| `get_all(ctx, name)` | `pub function` | Returns every value collected for a repeatable option. |
| `get_arg(ctx, index, default)` | `pub function` | Returns a positional value by position. |
| `get_arg_named(ctx, name, default)` | `pub function` | Returns a positional value by argument name. |
| `get_args(ctx)` | `pub function` | Returns every positional value in order. |
| `get_command(ctx)` | `pub function` | Returns the name of the matched command. |
| `get_command_path(ctx)` | `pub function` | Returns the full path of the matched command. |
| `has_command(ctx, name)` | `pub function` | Reports whether the given command was matched. |
| `has(ctx, name)` | `pub function` | Reports whether the value appeared on the command line. |
| `has_errors(ctx)` | `pub function` | Reports whether the parse collected diagnostics. |
| `get_errors(ctx)` | `pub function` | Returns the collected `CliError` values. |
| `help_requested(ctx)` | `pub function` | Reports whether `-h`/`--help` was supplied. |
| `version_requested(ctx)` | `pub function` | Reports whether `-V`/`--version` was supplied. |

### Usage Screens & Diagnostics

| Symbol | Visibility | Description |
|---|---|---|
| `format_help(app)` | `pub function` (`help` feature, default-on) | Renders the application usage screen. |
| `format_command_help(app, cmd)` | `pub function` (`help` feature, default-on) | Renders the usage screen of a command. |
| `format_version(app)` | `pub function` (`help` feature, default-on) | Renders the version banner. |
| `format_errors(errors)` | `pub function` (`help` feature, default-on) | Renders a diagnostic report. |
| `print_help(app)` | `pub function` (`help` feature, default-on) | Prints the application usage screen. |
| `print_command_help(app, cmd)` | `pub function` (`help` feature, default-on) | Prints the usage screen of a command. |
| `print_version(app)` | `pub function` (`help` feature, default-on) | Prints the version banner. |

### Error Model

| Symbol | Visibility | Description |
|---|---|---|
| `make_error(code, subject, message)` | `pub function` | Constructs a `CliError`. |
| `format_error(err)` | `pub function` | Renders a `CliError` as a diagnostic string. |
| `format_caught_error(err)` | `pub function` | Renders a value captured by `catch err`. |
| `CliError` | `pub struct` | Structured diagnostic (`code`, `subject`, `message`). |
| `CliResult` | `pub struct` | Outcome of a strict parse (`ok`, `context`, `error`). |

### Data Models

| Symbol | Visibility | Description |
|---|---|---|
| `CliApp` | `pub struct` | Application specification (name, description, version, options, arguments, commands). |
| `CliCommand` | `pub struct` | Command specification (name, aliases, options, arguments, subcommands). |
| `CliOption` | `pub struct` | Option specification (names, kind, default, required, choices, env variable). |
| `CliArgument` | `pub struct` | Positional argument specification (name, required, default, variadic). |
| `CliContext` | `pub struct` | Parsed execution context (flags, options, positionals, diagnostics). |
| `OptionKind` | `pub enum` | Option kinds (`Flag`, `Value`, `Multi`). |
| `CommandKind` | `pub enum` | Command kinds (`Root`, `Subcommand`). |
| `TokenKind` | `pub enum` | Token kinds recognised by the parser (`Inverted`, `Long`, `Short`, `Positional`). |
| `ErrorCode` | `pub enum` | Diagnostic codes (`UnknownOption`, `MissingValue`, `MissingRequired`, `InvalidChoice`, `MissingArgument`, `UnknownCommand`, `InvalidUsage`). |

### Fluent Methods

| Symbol | Visibility | Description |
|---|---|---|
| `CliApp.flag/option/multi_option/choice_option/required_option/env_option(...)` | `pub method` | Registers options on an application. |
| `CliApp.argument/variadic_argument(...)` | `pub method` | Registers positional arguments on an application. |
| `CliApp.command(cmd)` | `pub method` | Registers a command. |
| `CliApp.set_author/set_version(...)` | `pub method` | Updates the application metadata. |
| `CliApp.parse(args)` / `CliApp.run(args)` | `pub method` | Parses the given or the process arguments. |
| `CliCommand.flag/option/multi_option/choice_option/required_option/env_option(...)` | `pub method` | Registers options on a command. |
| `CliCommand.argument/variadic_argument(...)` | `pub method` | Registers positional arguments on a command. |
| `CliCommand.subcommand(cmd)` | `pub method` | Nests a command. |
| `CliContext.get_flag/flag/get_option/option/get_int/get_bool/get_all(...)` | `pub method` | Reads the resolved values. |
| `CliContext.get_arg/arg/get_arg_named/get_args/args(...)` | `pub method` | Reads the positional values. |
| `CliContext.get_command/command/get_command_path/has_command(...)` | `pub method` | Reads the command routing state. |
| `CliContext.has/has_errors/get_errors/errors(...)` | `pub method` | Reads the diagnostic state. |
| `CliContext.help_requested/version_requested(...)` | `pub method` | Reads the automatic switch state. |
| `CliResult.is_ok/is_error/message(...)` | `pub method` | Reads the outcome of a strict parse. |

> [!TIP]
> **Internal Helpers & Documentation:** Public symbols are documented with `##` Markdown docstrings, enabling automatic API documentation generation via `alya doc`. The helpers in `src/core/internal.alya` are not annotated with `pub` and stay encapsulated within the package.

### Behaviour Notes

- **Unknown input is reported, not ignored.** An option or a command that the specification does not declare produces a diagnostic (`UnknownOption` / `UnknownCommand`) instead of being silently accepted.
- **The command scope wins.** When a command re-declares a global option, the command definition is authoritative: the option is listed once in the usage screen, it is validated once (so no diagnostic is duplicated), and a value given *before* the command name is kept instead of being reset to the command default.
- **The command line wins over the environment.** An option registered with `env_option` reads its environment variable only when the option was not given on the command line. An environment provided value does not count as provided for `has()`.
- **Diagnostics are values, not strings.** `get_errors()` returns `CliError` values: `err.code` classifies the failure, `err.subject` names the offending option, argument, or command, and `err.message` explains it. Rebuild a diagnostic with `make_error(...)` before throwing it — a value read back from the untyped diagnostics array loses its struct identity when it crosses a `throw`.

---

## 🧪 Running Tests & Benchmarks

Run the automated test suite using `alya test`:

```bash
alya test
```

Exercise feature selection (the `help`, `strict`, and `env` slices are default-on):

```bash
alya test --features help
alya test --features strict
alya test --no-default-features
```

Generate static API documentation:

```bash
alya doc . -o docs --markdown
```

Run the benchmark suite:

```bash
alya run benches/bench_basic.alya
```

Run the example demo:

```bash
alya run examples/demo.alya
```

Check code formatting:

```bash
alya fmt . --check
```

Run static code linter:

```bash
alya lint . --check
```

### 💻 Developer Tooling & VS Code Integration

This package comes preconfigured with recommended workspace settings and tasks for **Visual Studio Code**:
- **LSP & Formatting**: Auto-formatting on save and real-time Language Server diagnostics via `alya-lang.vscode-alya`.
- **DAP Debugging**: Launch configurations in `.vscode/launch.json` ready for interactive step-debugging via `F5`.
- **Predefined Tasks**: Press `Ctrl+Shift+B` or run tasks (`Test`, `Lint`, `Format`, `Build Docs`) directly from the Command Palette.

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository and clone it locally
2. Install dependencies:
   ```bash
   alya install
   ```
3. Create your feature branch (`git checkout -b feature/my-feature`)
4. Verify tests and formatting before opening a PR:
   ```bash
   alya test
   alya fmt . --check
   alya lint . --check
   ```
5. Commit your changes (`git commit -m "feat: add feature"`) and open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
