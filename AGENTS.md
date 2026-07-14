# AGENTS.md

## Cursor Cloud specific instructions

`deckgym-core` is a Rust library + CLI that simulates Pokémon TCG Pocket games,
with optional Python bindings. There are two developable surfaces:

### Rust core (primary)
Standard commands are documented in `README.md` and `CONTRIBUTING.md`. Use the
CI feature flags for parity:

- Build: `cargo build --features "tui test-utils" --all-targets`
- Lint: `cargo fmt -- --check` and `cargo clippy --features "tui test-utils" -- -D warnings`
- Test: `cargo test --features "tui test-utils" --all-targets`
- Run CLI: `cargo run simulate example_decks/venusaur-exeggutor.txt example_decks/weezing-arbok.txt --num 1000 --players r,r -v`

Non-obvious caveats:
- The code uses trait-upcasting coercion (`dyn Trait` → `dyn Any`), which is only
  stable on **Rust >= 1.86**. The environment is pinned to the latest `stable`
  toolchain via `rustup default stable`; do not downgrade below 1.86 or the crate
  will fail to compile with `E0658` errors.
- Several `cargo test` targets are benchmarks/examples that print "Testing ... /
  Success" instead of the usual test harness output — this is expected, not a failure.

### Python bindings (secondary)
Built with `uv` + `maturin` (see `python/deckgym/README.md`). `uv` is installed at
`~/.local/bin` and is already on the login PATH.

- Set up venv (once): `uv sync --no-install-project` then `uv pip install --python .venv/bin/python maturin`
- Build/install the extension after any Rust change: `.venv/bin/maturin develop --features python`
- Test: `.venv/bin/pytest python/deckgym/tests`

Non-obvious caveats:
- The extension is a compiled Rust module; editing Rust does **not** hot-reload into
  Python. Re-run `maturin develop --features python` (or `uv run ...`, which rebuilds
  the project) after Rust changes before running Python tests.
- `.venv/` and `target/` are gitignored and persist in the VM snapshot.
