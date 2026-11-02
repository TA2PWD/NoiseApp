# [T-2] CI workflow

Milestone: M0 · Type: chore · Layer: all · Size: small · Status: todo

## Goal

Check format, lints and tests on every push and pull request, on both target OSes.

## Scope

- GitHub Actions matrix: `windows-latest`, `ubuntu-latest`.
- Steps: `cargo fmt --check`, `cargo clippy --workspace --all-targets -- -D warnings`, `cargo test --workspace`.
- Cargo build cache.

## Out of scope

- Frontend jobs (M5), golden test and model download (M3), release builds (M7).

## Acceptance criteria

- [ ] Runs on every push and pull request.
- [ ] A format error, a clippy warning or a failing test fails the run (shown once on a throwaway branch).
- [ ] Both jobs pass on the T-1 skeleton.

## Test plan

_Filled when the task moves to `planned`._

## Implementation plan

_Filled when the task moves to `planned`._

## Notes

None.
