# Product

<!-- impeccable:product-schema 1 -->

## Platform

library (Rust crate `book-finance-core` with two local binaries)

## Users

- The owner and the agents building the Finance Platform: they use this kernel to prove money
  and double-entry invariants before any finance logic moves out of the current services.
- The root repository's harnesses (`task test:finance-shadow`, `task test:finance-command`),
  which pipe sanitized `finance.v1` JSON into the `finance-v1-shadow` and
  `finance-command-pilot` binaries.

## Product Purpose

A small, pure, typed kernel for money and double-entry ledger rules: exact currency scale, no
floating point, balanced entries, and a read model that parses the live `finance.v1` summary
shape without touching any data store.

Success: every finance number that later moves into this kernel matches the current service
(shadow parity) and no invariant can be broken by a caller.

## Positioning

The first Rust boundary of the Finance Platform, kept deliberately pure: no database, no
network, no credentials, so it can be tested exhaustively (property tests, kernel purity test)
and embedded anywhere later.

## Operating Context

- Not deployed and never connected to production. GitHub Actions runs `scripts/check.sh`
  (tests, rustfmt, clippy with `-D warnings`), a build on the declared `rust-version`, and the
  shared security workflow.
- The only environment input is `FINANCE_CORE_COMMANDS_ENABLED=true` for the command pilot; any
  other value keeps the command disabled.
- Sentry does not apply: nothing runs as a service, and the binaries exit with an error code
  for their harness.
- DOCPACK verdict: `showcase`.

## Capabilities and Constraints

- Money with explicit currency and scale; double-entry ledger invariants.
- `finance.v1` summary parser and schema (`fixtures/finance-v1.schema.json`); the contract test
  fails if the schema, the fixture and the parser drift apart.
- `finance.reconciliation.preview` command: preview-only, idempotent in process, never writes a
  ledger or finance record.
- Out of scope: storage, HTTP APIs and any production cutover.

## Measures

- `scripts/check.sh` green on `main` (target: always).
- Shadow parity mismatches against the current finance API in the root harness (target: 0).
- Tests: 7 Rust test files (`tests/*_test.rs`); the Big Loop scorecard counts only Python,
  Go and JS/TS test names, so it reports this repository as missing tests.
