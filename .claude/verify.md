# Verify recipe (finance-ledger-core)

The `verify` skill follows this file. The checks are the ones CI runs.

## Checks

```bash
bash scripts/check.sh     # cargo test --locked, cargo fmt -- --check, cargo clippy -D warnings
```

Pure crate: offline, no env vars, no coverage gate. CI also runs `cargo test --locked` on the
MSRV (1.85): run `cargo +1.85 test --locked` when the change touches syntax or std APIs, or
`Cargo.toml` dependencies or `Cargo.lock` (a newer dependency can need a newer compiler while stable
still passes).

## End to end

Not a service. For money logic, add or run a property test (round-trip parse/format, `allocate`
sums to the input, balancing never crosses currencies) and show its case count.

`scripts/check.sh` never runs the binaries' `main`. When `src/bin/`, `src/shadow.rs` or
`src/command.rs` change, run them on real input (offline; the root repository's
`task test:finance-shadow` / `task test:finance-command` harnesses cover the same, `README.md`):

```bash
cargo run -q --locked --bin finance-v1-shadow < fixtures/finance-v1-summary.json   # prints parity counts
printf '{"command":"finance.reconciliation.preview","idempotency_key":"verify-001","summary":%s}' \
  "$(cat fixtures/finance-v1-summary.json)" > /tmp/cmd.json
FINANCE_CORE_COMMANDS_ENABLED=true cargo run -q --locked --bin finance-command-pilot < /tmp/cmd.json  # preview JSON, exit 0
for v in TRUE 1 yes ""; do FINANCE_CORE_COMMANDS_ENABLED=$v cargo run -q --locked --bin finance-command-pilot < /tmp/cmd.json; echo "$v -> $?"; done  # each exits 2
```

## Never

- Pointing anything here at production: the crate has no production connection and the cutover is
  not approved (`docs/gates/BOO-140-finance-api-boundary.md`).
