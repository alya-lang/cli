# cli

[![CI](https://github.com/alya-lang/cli/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/cli/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/cli?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fcli%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fcli%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

Modern, feature-rich command-line interface, argument parsing, flag validation, and subcommand routing toolkit for the Alya programming language.

---

## 🌟 Features

- ⚡ **High Performance**: Native compiled parser with sub-microsecond option lookups and minimal allocations.
- 🏳️ **Comprehensive Flag Support**:
  - Long flags: `--verbose`, `--debug`
  - Short flags: `-v`, `-d`
  - Clustered short flags: `-abc` (equivalent to `-a -b -c`)
  - Inverted boolean flags: `--no-color`, `--no-cache` (sets `color` / `cache` to 0)
- 🎛️ **Flexible Option Parsing**:
  - Key-value options: `--output=dist`, `--output dist`
  - Attached short options: `-oDist`, `-o dist`
  - Repeated / multi-value options: `-I dir1 -I dir2` (accumulates into array)
- 🛡️ **Validation & Constraints**:
  - Choice restrictions: e.g. `--format` allowed only in `["json", "yaml", "text"]`
  - Required options: automated reporting when mandatory options are omitted
  - Argument arity and type checks
- 🌳 **Subcommand Routing**:
  - Full hierarchical subcommands with isolated options and arguments (e.g. `forge build -r`, `forge test`)
  - Command aliases (e.g. `build` with alias `b`)
- 📄 **Automated Help & Version**:
  - Auto-generated, column-aligned help screens for applications and subcommands
  - Automatic `-h, --help` and `-V, --version` flag handling

---

## 📁 Project Architecture

```
cli/
├── alya.toml               # Package manifest
├── src/
│   ├── lib.alya            # High-level facade API
│   ├── types.alya          # CliApp, CliCommand, CliOption, CliArgument, CliContext
│   └── core/
│       ├── parser.alya     # Parsing engine, flag clustering, validation logic
│       ├── formatter.alya  # Help screen and version formatting
│       └── utils.alya      # String and array utilities
├── examples/
│   └── demo.alya           # Working demonstration CLI application
├── tests/
│   ├── test_basic.alya          # App creation, flags, options, double-dash
│   ├── test_flags_advanced.alya # Clustered flags, inverted flags, multi options
│   ├── test_commands.alya       # Subcommands, aliases, command-scoped flags
│   ├── test_validation.alya     # Choice validation, required options, errors
│   └── test_formatter.alya      # Help screen and version formatting tests
└── benches/
    └── bench_basic.alya    # Micro-benchmark suite
```

---

## 📦 Installation

Add `cli` to your project's `alya.toml`:

```toml
[dependencies]
cli = { git = "https://github.com/alya-lang/cli", branch = "main" }
```

Or install it directly with `alya`:

```bash
alya add cli --git https://github.com/alya-lang/cli --branch main
alya install
```

---

## 🚀 Quick Start

```alya
import "cli" as cli

function main()
    # 1. Create CLI application
    let app = cli::new("forge", "Alya build and testing tool", "1.0.0")

    # 2. Add global flags and options
    cli::app_add_flag(app, "-v, --verbose", "Enable verbose logs")
    cli::app_add_option(app, "-o, --output", "dist", "Output directory")

    # 3. Add subcommand
    let build_cmd = cli::command("build", "Compile package artifacts", ["b"])
    cli::cmd_add_flag(build_cmd, "-r, --release", "Build optimized release")
    cli::cmd_add_argument(build_cmd, "entrypoint", "Source file to compile", 0, "src/main.alya")
    cli::app_add_command(app, build_cmd)

    # 4. Parse command-line arguments
    let ctx = cli::parse_args(app)

    # 5. Check help or errors
    if cli::ctx_help_requested(ctx) == 1
        cli::print_help(app)
        return
    end

    if cli::ctx_has_errors(ctx) == 1
        say cli::cli_format_errors(cli::ctx_get_errors(ctx))
        exit(1)
    end

    # 6. Access parsed values
    if cli::ctx_has_command(ctx, "build") == 1
        let release = cli::ctx_get_flag(ctx, "release")
        let out_dir = cli::ctx_get_option(ctx, "output", "dist")
        let entry = cli::ctx_get_arg_named(ctx, "entrypoint", "src/main.alya")
        say "Building " + entry + " (release=" + str(release) + ") into " + out_dir
    end
end

main()
```

---

## 📖 API Reference

### Application & Command Construction

| Function | Arguments | Description |
|---|---|---|
| `cli::new(name, desc, ver)` | `name, desc = "", ver = "1.0.0"` | Creates a new `CliApp`. |
| `cli::app_set_author(app, author)` | `app, author` | Sets application author. |
| `cli::command(name, desc, aliases)` | `name, desc = "", aliases = []` | Creates a new `CliCommand`. |
| `cli::app_add_command(app, cmd)` | `app, cmd` | Registers a subcommand on the application. |

### Option & Flag Definition

| Function | Target | Description |
|---|---|---|
| `cli::app_add_flag(app, spec, desc)` | `CliApp` | Adds a boolean flag (`-v, --verbose`). |
| `cli::app_add_option(app, spec, def, desc)` | `CliApp` | Adds a string option with default value. |
| `cli::app_add_required_option(app, spec, desc)` | `CliApp` | Adds a required option. |
| `cli::app_add_multi_option(app, spec, desc)` | `CliApp` | Adds a repeatable option (e.g. `-I dir`). |
| `cli::app_add_choice_option(app, spec, def, choices, desc)` | `CliApp` | Adds an option restricted to specific choice strings. |
| `cli::app_add_argument(app, name, desc, req, def, var)` | `CliApp` | Adds a positional argument. |
| `cli::cmd_add_flag(cmd, spec, desc)` | `CliCommand` | Adds a flag scoped to the subcommand. |
| `cli::cmd_add_option(cmd, spec, def, desc)` | `CliCommand` | Adds an option scoped to the subcommand. |
| `cli::cmd_add_choice_option(cmd, spec, def, choices, desc)` | `CliCommand` | Adds a choice-restricted option to the subcommand. |
| `cli::cmd_add_argument(cmd, name, desc, req, def, var)` | `CliCommand` | Adds a positional argument to the subcommand. |

### Parsing & Context Queries

| Function | Arguments | Description |
|---|---|---|
| `cli::parse(app, raw_args)` | `app, args_array` | Parses argument array into `CliContext`. |
| `cli::parse_args(app)` | `app` | Parses current process command-line arguments. |
| `cli::ctx_get_flag(ctx, name)` | `ctx, name` | Returns 1 if flag was set, 0 otherwise. |
| `cli::ctx_get_option(ctx, name, def)` | `ctx, name, def = ""` | Returns option string value or fallback default. |
| `cli::ctx_get_multi_option(ctx, name)` | `ctx, name` | Returns array of string values for repeated option. |
| `cli::ctx_get_command(ctx)` | `ctx` | Returns the active subcommand name or `""`. |
| `cli::ctx_has_command(ctx, name)` | `ctx, name` | Returns 1 if subcommand matches, 0 otherwise. |
| `cli::ctx_get_arg(ctx, index, def)` | `ctx, index, def = ""` | Returns positional argument by index. |
| `cli::ctx_get_arg_named(ctx, name, def)` | `ctx, name, def = ""` | Returns positional argument by its defined name. |
| `cli::ctx_has_errors(ctx)` | `ctx` | Returns 1 if validation/syntax errors occurred, 0 otherwise. |
| `cli::ctx_get_errors(ctx)` | `ctx` | Returns array of error message strings. |
| `cli::ctx_help_requested(ctx)` | `ctx` | Returns 1 if `-h` or `--help` was encountered. |
| `cli::ctx_version_requested(ctx)` | `ctx` | Returns 1 if `-V` or `--version` was encountered. |

### Formatting & Screens

| Function | Arguments | Description |
|---|---|---|
| `cli::help(app, cmd = 0)` | `app, cmd = 0` | Returns formatted help screen as string. |
| `cli::print_help(app, cmd = 0)` | `app, cmd = 0` | Prints formatted help screen to stdout. |
| `cli::cli_format_version(app)` | `app` | Returns version string (e.g. `forge 1.0.0`). |
| `cli::cli_format_errors(errors)` | `errors` | Returns newline-separated error block. |

---

## 🧪 Running Tests & Benchmarks

Run the test suite:

```bash
alya test
```

Run micro-benchmarks:

```bash
alya run benches/bench_basic.alya
```

Run the interactive demo:

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

---

## 🤝 Contributing

1. Fork the repository and clone locally
2. Install dependencies:
   ```bash
   alya install
   ```
3. Run tests and verify code formatting:
   ```bash
   alya test
   alya fmt . --check
   ```
4. Commit your changes and open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.