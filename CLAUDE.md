# CLAUDE.md

yo, welcome to the zed-claude code integration project. this guide helps you stay focused and write solid rust code.

## Quick Start Checklist

before diving in:

1. [ ] **model check**: mention which model you are (e.g., "working with claude-3-opus")
2. [ ] **read the docs**: skim through the project knowledge files
3. [ ] **explain first**: describe what you're about to do before coding
4. [ ] **ask permission**: check if the approach sounds good before implementing
5. [ ] **test everything**: run cargo test after changes
6. [ ] **commit often**: make atomic commits with clear messages

## Project Overview

we're building a rust-based lsp bridge that connects zed editor to claude code. think of it as a translator between zed's expectations and claude's capabilities.

### architecture tldr

```
zed extension (wasm) → lsp bridge server (rust) → claude code process
```

- **zed extension**: webassembly module that spawns our bridge
- **lsp bridge**: rust server pretending to be a language server
- **claude process**: managed subprocess running claude in pipe mode

## Code Quality Standards

### rust specifics

- use `tokio` for async runtime
- prefer `miette` for error handling in apps, `thiserror` for libraries
- `tower-lsp` or `lsp-server` for lsp implementation
- keep dependencies minimal - we're not building a spaceship

### validation commands

primary loop (run these constantly):

```bash
cargo check       # type checking - this is what rust-analyzer runs
cargo test        # run all tests
```

**note**: since you can't pipe live output from long-running commands, stick to `cargo check` and `cargo test` for your main feedback loop. they're quick and give you what you need.

with just commands:

```bash
just fmt          # format code
just test         # test with file watching
just w            # watch, compile, and run on changes
```

before committing:

```bash
nix flake check   # final validation
```

if something fails, fix it before moving on. no shortcuts.

## Implementation Phases

### phase 1: lsp bridge foundation (current focus)

- [ ] basic lsp server with `tower-lsp`
- [ ] handle initialize/completion requests
- [ ] spawn and manage claude subprocess
- [ ] parse streaming json responses

### phase 2: zed extension

- [ ] webassembly extension that spawns bridge
- [ ] proper manifest configuration
- [ ] handle api key management

### phase 3: context extraction

- [ ] extract editor state via lsp
- [ ] build rich context from diagnostics/symbols
- [ ] optimize for token limits

### phase 4: error handling & recovery

- [ ] circuit breaker pattern
- [ ] process restart logic
- [ ] graceful degradation

### phase 5: performance & polish

- [ ] lru caching for context
- [ ] streaming optimizations
- [ ] installation scripts

## Project Structure

```
zed-claude-code/
├── Cargo.toml           # workspace root
├── bridge/              # lsp bridge server
│   ├── src/
│   │   ├── main.rs      # server entry point
│   │   ├── claude.rs    # claude process management
│   │   ├── context.rs   # context extraction
│   │   └── lsp.rs       # lsp implementation
│   └── Cargo.toml
├── extension/           # zed extension
│   ├── src/
│   │   └── lib.rs       # extension entry
│   ├── extension.toml   # manifest
│   └── Cargo.toml
└── flake.nix           # nix development environment
```

## Common Patterns

### error handling

```rust
// for recoverable errors
use miette::{IntoDiagnostic, Result};

// for library errors
use thiserror::Error;
use miette::Diagnostic;

#[derive(Error, Debug, Diagnostic)]
enum BridgeError {
    #[error("claude process died: {0}")]
    ProcessDied(String),
}
```

### async subprocess management

```rust
use tokio::process::Command;
use tokio::io::{AsyncBufReadExt, BufReader};

let mut child = Command::new("claude")
    .args(&["-p", "--output-format", "stream-json"])
    .stdin(Stdio::piped())
    .stdout(Stdio::piped())
    .spawn()?;
```

### lsp implementation

```rust
use tower_lsp::{jsonrpc::Result, lsp_types::*, Client, LanguageServer};

#[tower_lsp::async_trait]
impl LanguageServer for ClaudeCodeBridge {
    async fn completion(&self, params: CompletionParams) -> Result<Option<CompletionResponse>> {
        // your logic here
    }
}
```

## Testing Strategy

- unit tests for core logic
- integration tests for lsp protocol
- mock claude responses for deterministic testing
- use `tokio::test` for async tests

```rust
#[tokio::test]
async fn test_claude_process_spawning() {
    // test implementation
}
```

## Security Considerations

- **never** log api keys
- use system keychain for storage
- validate all file paths
- run claude with restricted permissions
- implement rate limiting

## Debugging Tips

- use `tracing` for structured logging
- `RUST_LOG=debug` for verbose output
- test lsp with `lsp-test` or similar tools
- mock claude responses during development

## Common Pitfalls

- forgetting to handle subprocess termination
- not implementing proper backpressure for streaming
- assuming lsp requests come in order (they don't)
- hardcoding paths or api keys

## Quick Reference

### useful crates

- `tower-lsp`: lsp server implementation
- `tokio`: async runtime
- `serde_json`: json parsing
- `tracing`: logging
- `miette`/`thiserror`: error handling
- `keyring`: secure credential storage

### claude code cli flags

```bash
claude -p                              # pipe mode
--output-format stream-json            # streaming responses
--system-prompt "..."                  # set system prompt
--allowedTools "View,Edit,GrepTool"    # tool permissions
--dangerously-skip-permissions         # for automation
```

## Contributing

- keep commits atomic and descriptive
- follow conventional commit format: `<type>[optional scope]: <description>`
  - types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`
  - example: `feat(bridge): add streaming json parser`
- run all validation commands before committing
- update this file when adding major features
- ask questions if architecture seems unclear

remember: we're building a bridge, not a cathedral. keep it simple, make it work, then make it better.

---

*last updated: when you started reading this*
