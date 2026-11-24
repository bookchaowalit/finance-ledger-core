# Review guide (finance-ledger-core)

The Codex Stop-hook review reads this file from the default branch, never from the change under
review. Owner-written; keep it under 16 KiB.

## What failure looks like here

Treat these as `[P0]`/`[P1]`:

- **Money**: `i128` minor units, never float; JSON amounts never through f64 (`arbitrary_precision`);
  exponent notation (`1e3`) and a trailing dot (`1.`) rejected (`README.md` invariants).
- **Ledger rules**: no arithmetic across currencies, postings positive, every entry balances in one
  currency, duplicate transaction ids refused in a book (`src/lib.rs`).
- **Allocation**: `allocate` sums exactly to the original amount.
- **Purity**: the kernel gains no network, DB, env or filesystem dependency
  (`tests/kernel_purity_test.rs`); the public API stays as pinned in
  `fixtures/gates/finance-api-boundary.json`.
- **Command pilot** runs only when `FINANCE_CORE_COMMANDS_ENABLED=true` exactly.

## Severity

- `[P0]` a wrong amount, an unbalanced entry accepted, float anywhere on the money path.
- `[P1]` a valid input refused or the public API changed without its boundary fixture.
- `[P2]` edge case, or changed behaviour without a test.
- `[P3]` nit. **At most 3 nits.**

## Skip

- `DOCPACK/pdf`.
- `Cargo.lock`: not skipped; an update must be deliberate, review its diff.
- `fixtures/finance-v1.schema.json` and `fixtures/finance-v1-summary.json`: not skipped. They are the
  `finance.v1` contract (`README.md`); review them against the parser, since
  `tests/contract_schema_test.rs` checks field names and constants, not value types.

## Evidence

Every finding gives `file:line` and input -> wrong result. No finding without evidence from the diff
or the code it calls. Start the summary with the count, e.g. `2 findings (1 P1, 1 P3)`.
