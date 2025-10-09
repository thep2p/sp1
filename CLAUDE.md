# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

SP1 is a high-performance zero-knowledge virtual machine (zkVM) that can prove the execution of arbitrary Rust programs. It enables developers to write ZK proofs in standard Rust code, making zero-knowledge technology accessible without requiring specialized cryptographic knowledge.

## Common Development Commands

### Building the Project

```bash
# Build the entire workspace
cargo build --all

# Build with release optimizations
cargo build --release

# Build with all features
cargo build --all-features

# Install SP1 CLI from source
cd crates/cli
cargo install --locked --force --path .

# Install SP1 toolchain
cargo run -p sp1-cli -- prove install-toolchain
```

### Running Tests

```bash
# Run all tests in release mode with optimizations
cargo test --release --features native-gnark --workspace --exclude sp1-verifier

# Test specific package
cargo test --package <package-name> --release

# Test SP1 verifier with special features
cargo test --release --package sp1-verifier -F ark

# Test with debug features
RUST_LOG=info RUST_BACKTRACE=1 cargo test <test-name> --release --features debug -- --nocapture

# Run tests in the core crate
cd core && cargo test
```

### Working with SP1 Programs

```bash
# Create new SP1 project
cargo prove new <project-name>

# Build SP1 program (run from program directory)
RUSTFLAGS="-C passes=loweratomic -C link-arg=-Ttext=0x00200800 -C panic=abort" cargo prove build

# Execute SP1 program
cargo prove execute --elf <path-to-elf>

# Generate proof
cargo prove prove --elf <path-to-elf>

# Trace program execution
cargo prove trace --elf <path-to-elf> --trace <output-path>
```

### Code Quality Checks

```bash
# Format code (uses nightly)
cargo +nightly fmt --all

# Check formatting
cargo +nightly fmt --all -- --check

# Run clippy
cargo clippy --all-features --all-targets -- -D warnings -A incomplete-features

# Check compilation without building
cargo check --all-targets --all-features
```

## High-Level Architecture

### Core Components

1. **crates/cli** - Command-line interface (`cargo prove`) for building, executing, and proving SP1 programs
2. **crates/core** - Core execution engine consisting of:
   - `executor` - RISC-V interpreter and execution logic
   - `machine` - Core zkVM implementation with AIR constraints
3. **crates/prover** - Proof generation system using Plonky3 backend
4. **crates/sdk** - SDK for integrating SP1 into applications
5. **crates/recursion** - Recursive proof composition system:
   - `circuit` - Circuit definitions
   - `compiler` - Recursion compiler
   - `gnark-ffi` - FFI bindings for Groth16/PLONK proving
6. **crates/stark** - STARK proof system implementation
7. **crates/zkvm** - SP1 zkVM runtime libraries:
   - `entrypoint` - Program entry point
   - `lib` - Core zkVM library with precompiles
8. **crates/verifier** - On-chain proof verification contracts

### Program Structure

SP1 programs consist of two parts:
- **Program**: RISC-V binary compiled from Rust that runs inside the zkVM
- **Script**: Host program that executes the zkVM and generates proofs

Example structure:
```
examples/fibonacci/
├── program/        # zkVM program (compiled to RISC-V)
│   ├── Cargo.toml
│   └── src/main.rs
└── script/         # Host script
    ├── Cargo.toml
    └── src/main.rs
```

### Key Concepts

- **Precompiles**: Accelerated operations (e.g., BN254, BLS12-381, Keccak) implemented as syscalls
- **Shards**: Programs are split into execution shards for parallel proving
- **Recursion**: Multiple proofs can be aggregated using recursive verification
- **Network Prover**: Optional remote proving service for resource-intensive proofs

## Development Workflow

### Setting Up a Development Environment

1. Install Rust (MSRV: 1.79)
2. Install Go (required for gnark-ffi)
3. Install SP1 toolchain: `cargo run -p sp1-cli -- prove install-toolchain`
4. Install CLI: `cd crates/cli && cargo install --locked --force --path .`

### Testing Changes

When modifying core components:
1. Run relevant unit tests in the modified crate
2. Run integration tests to ensure compatibility
3. Test with example programs in `examples/` directory
4. Check that CI passes: format, clippy, tests

### Debugging Tips

- Use `--features debug` for detailed constraint failure information
- Set `RUST_LOG=info` for verbose logging
- Set `SP1_DEV=1` to use development mode settings
- Use `RUST_BACKTRACE=1` for detailed error traces

## Important Files and Directories

- `Cargo.toml` - Workspace configuration
- `crates/` - Main source code organized by component
- `examples/` - Example SP1 programs demonstrating features
- `patch-testing/` - Test patches for dependency updates
- `.github/workflows/` - CI/CD configuration
- `audits/` - Security audit reports

## Branch Structure

- `main` - Stable release branch
- `dev` - Development branch (PR target for new features)

## Release Process

Releases are managed via `release-plz`:
1. Changes merged to `dev` trigger PR creation
2. PR merge publishes to crates.io
3. After publish, `dev` is merged to `main`