# MoonBit Claude Code Skills

Claude Code skills for MoonBit language development, based on the official [MoonBit documentation](https://github.com/moonbitlang/moonbit-docs).

## Skills Overview

| Skill | Description | Trigger |
|-------|-------------|---------|
| `moonbit-language` | Core language syntax, data types, control flow, error handling, methods & traits | Writing `.mbt` files, asking about syntax |
| `moonbit-testing` | Test framework, unit tests, snapshot tests, blackbox/whitebox tests | Writing tests, running `moon test` |
| `moonbit-ffi` | FFI interop with JavaScript, C, Wasm - external types, functions, callbacks | FFI development, external function declarations |
| `moonbit-toolchain` | `moon` build tool, project management, package management, formatting | Using `moon` commands, project setup |

## Installation

### Option 1: Copy to Claude Skills Directory

```bash
# Copy skills to Claude's skills directory
cp -r moonbit-* ~/.claude/skills/
```

### Option 2: Symbolic Links

```bash
# Create symbolic links
ln -s /path/to/moonbit-claude-skills/moonbit-language ~/.claude/skills/moonbit-language
ln -s /path/to/moonbit-claude-skills/moonbit-testing ~/.claude/skills/moonbit-testing
ln -s /path/to/moonbit-claude-skills/moonbit-ffi ~/.claude/skills/moonbit-ffi
ln -s /path/to/moonbit-claude-skills/moonbit-toolchain ~/.claude/skills/moonbit-toolchain
```

## Usage

Once installed, the skills will be automatically triggered when you:

- Write or modify MoonBit code (`.mbt` files)
- Ask about MoonBit syntax or features
- Write or run tests
- Work with FFI interop
- Use `moon` commands

You can also explicitly invoke them:

```
/moonbit-language    # Language reference
/moonbit-testing     # Testing guide
/moonbit-ffi         # FFI interop guide
/moonbit-toolchain   # Toolchain commands
```

## Skill Contents

### moonbit-language
- Built-in data types (Unit, Boolean, numbers, String, Tuple, Option, Result, Array, Map)
- Functions (named parameters, optional parameters, arrow functions, partial application)
- Control structures (if/match, while, for, list comprehensions, defer)
- Custom data types (Struct, Enum, Type Alias)
- Pattern matching (basic, array, guard, is expression)
- Error handling (raise, try-catch, Result conversion)
- Methods & Traits (definition, implementation, built-in traits)
- Generics
- Special syntax (pipe, cascade, spread, regex)

### moonbit-testing
- Test blocks and assertions
- Snapshot tests (Show, JSON, generic)
- Blackbox vs whitebox testing
- Test configuration
- Running tests
- Test coverage
- Best practices

### moonbit-ffi
- Backend support (Wasm, Wasm GC, JS, C, LLVM)
- External type declarations
- External function declarations
- ABI type mappings
- Callback handling (FuncRef vs closures)
- Exporting functions
- Lifetime management (reference counting, ownership)

### moonbit-toolchain
- Core commands (new, build, check, run, test, fmt, doc)
- Common options
- Project structure (workspace, module config, package config)
- Package management
- Target platforms
- Debugging and diagnostics

## License

This project is based on the [MoonBit documentation](https://github.com/moonbitlang/moonbit-docs) and is subject to its licensing terms:

- **Code snippets**: Apache License 2.0
- **Prose content (after July 4, 2024)**: CC BY-SA 4.0

See [LICENSE](LICENSE) for details.

## Attribution

- **Original source**: [moonbitlang/moonbit-docs](https://github.com/moonbitlang/moonbit-docs)
- **MoonBit language**: [moonbitlang/moonbit](https://github.com/moonbitlang/moonbit)
- **MoonBit package registry**: [mooncakes.io](https://mooncakes.io)

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## Acknowledgments

Thanks to the MoonBit team for creating excellent documentation that made these skills possible.
