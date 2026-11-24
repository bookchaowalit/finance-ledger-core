# Verify recipe (finance-ledger-core)

The `verify` skill follows this file. The checks are the ones CI runs.

## Checks

```bash
bash scripts/check.sh     # cargo test --locked, cargo fmt -- --check, cargo clippy -D warnings
```

Pure crate: offline, no env vars, no coverage gate. CI also runs `cargo test --locked` on the
MSRV (1.85): use that toolchain when the change touches syntax or std APIs.

## End to end

Not a service. For money logic, add or run a property test (round-trip parse/format, `allocate`
sums to the input, balancing never crosses currencies) and show its case count.

## Never

- Pointing anything here at production: the crate has no production connection and the cutover is
  not approved (`docs/gates/BOO-140-finance-api-boundary.md`).
