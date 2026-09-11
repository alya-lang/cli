# cli

[![CI](https://github.com/alya-lang/cli/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/cli/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/cli?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fcli%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fcli%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

Modern command-line interface, argument parsing, flag handling, and subcommand router for Alya

---

## 🌟 Features

- ⚡ **Lightweight & Fast**: Built for speed with minimal overhead
- 🧩 **Modular Architecture**: Multi-module design supporting flat modules (`types.alya`) and subfolder hierarchies (`core/formatter.alya`)
- 🛡️ **Reliable & Typed**: Explicit struct definitions and clean namespaced APIs
- 🧪 **Well Tested**: Comprehensive test suite with standard assertions

---

## 📁 Project Architecture

```
cli/
├── alya.toml               # Package manifest
├── src/
│   ├── lib.alya            # Public API facade
│   ├── types.alya          # Data structures & struct definitions
│   └── core/               # Subdirectory module hierarchy (optional for larger packages)
│       └── formatter.alya  # Domain formatting logic & internal helpers
├── examples/
│   └── demo.alya           # Runnable usage examples
├── tests/
│   └── test_basic.alya     # Automated test suite
└── benches/
    └── bench_basic.alya    # Micro-benchmarks
```

> [!NOTE]
> Modules can be structured flat inside `src/` (e.g. `src/types.alya`) or grouped into subdirectories (e.g. `src/core/formatter.alya`). Relative imports like `import "../types.alya"` or `import "./core/formatter.alya"` are resolved relative to the importing file and deduplicated transitively.

---

## 📦 Installation

Add `cli` to the `[dependencies]` section in your `alya.toml`:

```toml
[dependencies]
cli = { git = "https://github.com/alya-lang/cli", tag = "v0.1.0" }
```

Or install it directly using the Alya package CLI:

```bash
alyac add cli --git https://github.com/alya-lang/cli --tag v0.1.0
alyac install
```

---

## 🚀 Quick Start

```alya
import "cli" as pkg

function main()
    # Basic facade call
    let greeting = pkg::hello("Alya")
    say greeting

    # Struct construction and domain helpers
    let cfg = pkg::new_config("Community", 2)
    say "Target: " + cfg.name
    say "Formatted: " + pkg::core_format_custom(cfg)
end

main()
```

---

## 📖 API Reference

| Function | Arguments | Returns | Description |
|---|---|---|---|
| `hello(name)` | `name = "World"` | `string` | Returns a friendly greeting message. |
| `new_config(name, count)` | `name = "World", count = 1` | `CliConfig` | Constructs a new configuration struct. |
| `core_format_greeting(name)` | `name` | `string` | Core formatter producing `Hello, {name}!`. |
| `core_format_custom(config)` | `config: CliConfig` | `string` | Formats greeting using prefix and name from config. |

---

## 🧪 Running Tests & Benchmarks

Run the test suite using `alyac`:

```bash
alyac run tests/test_basic.alya
```

Run the benchmark suite:

```bash
alyac run benches/bench_basic.alya
```

Run the example demo:

```bash
alyac run examples/demo.alya
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository and clone it locally
2. Install dependencies:
   ```bash
   alyac install
   ```
3. Create your feature branch (`git checkout -b feature/my-feature`)
4. Verify tests and formatting before opening a PR:
   ```bash
   alyac test
   alyac fmt . --check
   ```
5. Commit your changes (`git commit -m "feat: add feature"`) and open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.