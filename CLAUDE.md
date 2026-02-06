# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kulala.nvim is a REST client interface plugin for Neovim, written in Lua. It supports HTTP, gRPC, GraphQL, and WebSocket protocols with IntelliJ HTTP Client compatibility. The name "kulala" is Swahili for "rest" or "relax."

Requires Neovim 0.10.0+ and cURL.

## Common Commands

### Testing
```bash
make test                              # Run full test suite
make watch                             # Watch Lua files, re-run tests on change
make watch_tag tag=<tag>               # Watch with tag filtering
./scripts/tests.sh run                 # Run tests directly
./scripts/tests.sh run "<filter>"      # Run tests matching a filter string
```

Tests use Busted via lazy.nvim's minit framework. Test entry point is `tests/minit.lua`. Test specs live in `tests/functional/`.

### Linting and Formatting
```bash
./scripts/lint.sh check-code           # Check Lua formatting with stylua
./scripts/lint.sh check-code <file>    # Check specific file
stylua .                               # Auto-format all Lua files
```

### Documentation
```bash
make vimdocs                           # Generate vim help docs
./scripts/lint.sh check-docs           # Check docs prose with vale
```

## Code Style

- **Formatter:** stylua — 2-space indent, 120 column width, Unix line endings, double quotes preferred, `sort_requires` enabled (see `stylua.toml`)
- **Linter:** luacheck — lua51+nvim standard (see `.luacheckrc`)
- **Editor:** 2-space indent, UTF-8, final newline (see `.editorconfig`)

## Architecture

### Entry Point
`lua/kulala/init.lua` — Exports the public API. `setup(config)` initializes config and autogroups.

### Core Modules (`lua/kulala/`)

| Module | Purpose |
|--------|---------|
| `cmd/init.lua` | Request execution, queue management, response handling |
| `cmd/lsp.lua` | Built-in LSP server (completions, diagnostics, code actions, hover) |
| `cmd/oauth.lua` | OAuth2 authentication flows |
| `cmd/export.lua` | Export to Postman/other formats |
| `parser/document.lua` | Parses HTTP files into document structure |
| `parser/request.lua` | Parses individual HTTP requests |
| `parser/treesitter.lua` | Tree-sitter integration for the `kulala_http` grammar |
| `parser/env.lua` | Environment variable resolution |
| `parser/curl.lua` | Generates cURL commands from parsed requests |
| `parser/scripts/` | Pre/post-request script engines (JavaScript and Lua) |
| `config/defaults.lua` | All default configuration options |
| `config/keymaps.lua` | Keymap definitions |
| `ui/init.lua` | Main UI — response display, split/float windows |
| `ui/float.lua` | Floating window implementation |
| `formatter/formatter.lua` | Response formatting by content type |
| `db/` | Persistent state management |
| `globals/` | Constants and version |
| `utils/fs.lua` | File system utilities |

### Request Processing Pipeline
1. Parse HTTP file via Tree-sitter (`parser/document.lua`, `parser/request.lua`)
2. Resolve variables and run pre-request scripts
3. Generate and execute cURL command asynchronously
4. Parse response, run post-request scripts
5. Process assertions, update UI with results

### Queue System
Requests are processed through a FIFO queue in `cmd/init.lua` with pause/resume support. `run_all()` queues every request in a file; `run()` queues the request under the cursor.

### LSP Server
The plugin ships its own LSP server (`cmd/lsp.lua`) providing completions, diagnostics, symbols, code actions, and hover for `.http`/`.rest` files. Completion sources are in `cmd/lsp_sources.lua`.
