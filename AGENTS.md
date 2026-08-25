# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## The Golden Rule

When unsure about implementation details, ALWAYS ask the developer.

## Project Overview

`tobj` is a lightweight OBJ loader library inspired by tinyobjloader. It provides simple and efficient loading of Wavefront OBJ files and their associated MTL material files, returning loaded models and materials as Rust data structures.

## Build, Test, and Development Commands

```bash
# Build the library (debug mode - default for development)
cargo build

# Build with all features enabled
cargo build --all-features

# Run tests
cargo test

# Run tests with all features
cargo test --all-features

# Run a specific test
cargo test test_name

# Build and run the print_mesh example
cargo run --example print_mesh -- path/to/file.obj

# Format code
cargo fmt

# Check formatting (CI requirement)
cargo fmt -- --check

# Run clippy linter
cargo clippy --all-targets --all-features

# Generate documentation
cargo doc --open

# Build in release mode (ONLY when explicitly requested)
cargo build --release
```

## Architecture and Key Components

### Core Library Structure

- **Main Module** (`src/lib.rs`): Contains all core functionality including:
  - `Model` and `Mesh` structs for representing loaded geometry
  - `Material` struct for MTL material properties
  - Loading functions: `load_obj`, `load_obj_buf`, and async variants
  - `LoadOptions` for configuring loading behavior (triangulation, single indexing, etc.)

### Key Data Structures

- **Mesh**: Contains vertex positions, normals, texture coordinates, colors, and indices
  - Positions stored as flat `Vec<f32>` or `Vec<f64>` (with `use_f64` feature)
  - Supports both single and separate indexing for positions/normals/texcoords
  - Face arities track polygon sizes for non-triangulated meshes

- **Material**: Standard MTL properties plus unknown parameter storage in HashMap
  - Ambient, diffuse, specular colors
  - Texture maps for various properties
  - Custom parameters preserved in `unknown_param` HashMap

### Feature Flags

- **Default**: `ahash` for faster hashing
- **`merging`**: Merge identical vertices (requires beta/nightly for const generics)
- **`reordering`**: Reorder normal and texture coordinate indices
- **`async`**: Async loading support with custom material loader
- **`futures`**: Async with futures-lite AsyncRead traits
- **`tokio`**: Async with tokio AsyncRead traits
- **`use_f64`**: Use f64 instead of f32 for vertex data

## Important Implementation Details

### Memory Efficiency
- Flat arrays for vertex data minimize allocations
- Optional features to avoid unnecessary dependencies
- Configurable indexing strategies

### Triangulation
- Only supports trivial fan triangulation
- Complex polygons should be pre-triangulated in modeling software

### Testing
- Unit tests in `src/tests.rs`
- Test files in repository for validation
- CI runs tests on Linux, Windows, and macOS

## Coding Style & Naming Conventions

### Guidelines

- **CRITICAL: ALWAYS run `cargo test --all-features` and ensure the code compiles and tests pass WITHOUT ANY WARNINGS BEFORE committing!** Never commit code that doesn't build, has failing tests, or produces warnings. The `--all-features` flag is essential as it tests all feature combinations.
  - First run: `cargo test --all-features` to ensure everything compiles and passes without warnings
  - Then run: `cargo fmt` to format the code
  - Then run: `cargo clippy --all-targets --all-features` and fix any issues
  - Finally run: `cargo test --all-features` one more time to verify everything is clean
  - Only then commit the changes when there are ZERO warnings and all tests pass

- **CRITICAL: Address ALL warnings before EVERY commit!** This includes:
  - Unused imports, variables, and functions
  - Dead code warnings
  - Deprecated API usage
  - Type inference ambiguities
  - Any clippy warnings or suggestions
  - Never use `#[allow(warnings)]` or similar suppressions without explicit user approval

- DO NOT change any public-facing API without presenting a change proposal to the user first, including a rationale and getting permission to do so after.

- Follow standard Rust style: four-space indentation, `snake_case` for modules/functions, and `CamelCase` for types.

- Write idiomatic and canonical Rust code. Avoid patterns common in imperative languages that can be expressed more elegantly in Rust.

- PREFER functional style over imperative style when appropriate.

- AVOID unnecessary allocations, conversions, copies.

- AVOID using `unsafe` code unless absolutely necessary.

- Keep public APIs documented with `///` comments.

- Run `cargo fmt --all` before committing; the project assumes rustfmt defaults.

- Lint with `cargo clippy --all-targets --all-features` and address warnings.

### Naming Conventions (Rust API Guidelines)
- **Casing**: `UpperCamelCase` for types/traits/variants; `snake_case` for functions/methods/modules/variables; `SCREAMING_SNAKE_CASE` for constants/statics.
- **Conversions**: `as_` for cheap borrowed→borrowed; `to_` for expensive conversions; `into_` for ownership-consuming conversions.
- **Getters**: No `get_` prefix (use `width()` not `get_width()`), except for unsafe variants like `get_unchecked()`.
- **Iterators**: `iter()` for &T, `iter_mut()` for &mut T, `into_iter()` for T by value.

## What AI Must NEVER Do

1. **Never modify test files** - Tests encode human intent
2. **Never change API contracts** - This is a library used by many projects
3. **Never commit secrets** - Use environment variables
4. **Never assume business logic** - Always ask
5. **Never change expected test outputs** - These are the ground truth for tests

Remember: This is a library crate - backward compatibility and API stability are critical. When in doubt, choose the boring, stable solution.

## Documentation Guidelines

- All code comments MUST end with a period.
- All doc comments should also end with a period unless they're headlines.
- All list items in documentation MUST be complete sentences that end with a period.
- All comments must be on their own line. Never put comments at the end of a line of code.
- All references to types, keywords, symbols etc. MUST be enclosed in backticks: `struct` `Foo`.
- For each part of the docs, every first reference to a type that is NOT the item itself being described MUST be linked: [`Foo`].

## Writing Instructions For User Interaction And Documentation

- Be concise.
- Use simple sentences. But feel free to use technical jargon.
- Do NOT overexplain basic concepts. Assume the user is technically proficient.
- Maintain a neutral viewpoint.