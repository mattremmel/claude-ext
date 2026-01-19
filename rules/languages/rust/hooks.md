# Hooks Configuration - Rust

## PreToolUse Hooks

### tmux Reminder
Suggests tmux for long-running Cargo commands:
- `cargo build` (especially with `--release`)
- `cargo test`
- `cargo run`
- `cargo install`
- `cargo doc`

## PostToolUse Hooks

### rustfmt Auto-Format
Runs `rustfmt` on Rust files after editing:
- `.rs` files

Configuration in `rustfmt.toml`:

```toml
# rustfmt.toml
edition = "2024"
max_width = 100
tab_spaces = 4
use_small_heuristics = "Default"
reorder_imports = true
reorder_modules = true
group_imports = "StdExternalCrate"
```

Run manually:

```bash
# Format single file
rustfmt src/main.rs

# Format entire project
cargo fmt

# Check formatting without changing files
cargo fmt --check
```

### cargo check Integration
Runs `cargo check` after editing `.rs` files for fast type checking without full compilation:

```bash
# Fast type check (no code generation)
cargo check

# Check all targets including tests
cargo check --all-targets

# Check with all features
cargo check --all-features
```

### Clippy Linting
Runs Clippy after editing for lint warnings:

```bash
# Run Clippy
cargo clippy

# Treat warnings as errors (CI mode)
cargo clippy -- -D warnings

# Fix auto-fixable lints
cargo clippy --fix
```

Configuration in `Cargo.toml`:

```toml
[lints.clippy]
# Pedantic lints (opt-in, more strict)
pedantic = "warn"

# Allow specific lints when needed
module_name_repetitions = "allow"
```

Or in `clippy.toml`:

```toml
# clippy.toml
cognitive-complexity-threshold = 25
too-many-arguments-threshold = 10
```

### println!/dbg! Detection
Warns when debug macros are found in edited files:

```rust
// AVOID: Debug macros in production code
println!("Debug: {:?}", data);
dbg!(value);
eprintln!("Error info: {}", msg);

// PREFER: Structured logging with tracing
use tracing::{debug, info, error};

debug!(?data, "Processing data");
info!(user_id = %id, "User logged in");
error!(?err, "Operation failed");
```

### tracing Configuration

```rust
use tracing_subscriber::{layer::SubscriberExt, util::SubscriberInitExt};

fn init_tracing() {
    tracing_subscriber::registry()
        .with(
            tracing_subscriber::EnvFilter::try_from_default_env()
                .unwrap_or_else(|_| "info".into()),
        )
        .with(tracing_subscriber::fmt::layer())
        .init();
}
```

## Stop Hooks

### Debug Macro Audit
Before session ends, checks all modified `.rs` files for debug macros that should be removed before committing:

```bash
# Find println! statements
grep -rn "println!" src/

# Find dbg! macros
grep -rn "dbg!" src/

# Find all debug macros
grep -rn -E "(println!|dbg!|eprintln!)" src/
```

## Cargo Commands Reference

### Build Commands

```bash
# Debug build (fast compile, slow runtime)
cargo build

# Release build (slow compile, fast runtime)
cargo build --release

# Build specific binary
cargo build --bin my_binary

# Build with all features
cargo build --all-features
```

### Test Commands

```bash
# Run all tests
cargo test

# Run specific test
cargo test test_user_creation

# Run tests with output
cargo test -- --nocapture

# Run ignored tests
cargo test -- --ignored

# Run doc tests only
cargo test --doc
```

### Run Commands

```bash
# Run default binary
cargo run

# Run with arguments
cargo run -- --config config.toml

# Run specific binary
cargo run --bin server

# Run in release mode
cargo run --release
```

### Documentation

```bash
# Build documentation
cargo doc

# Build and open in browser
cargo doc --open

# Include private items
cargo doc --document-private-items
```

## CI Integration

```yaml
# .github/workflows/rust.yml
name: Rust CI
on: [push, pull_request]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with:
          components: rustfmt, clippy

      - name: Check formatting
        run: cargo fmt --check

      - name: Clippy
        run: cargo clippy -- -D warnings

      - name: Check
        run: cargo check --all-targets

      - name: Test
        run: cargo test
```
