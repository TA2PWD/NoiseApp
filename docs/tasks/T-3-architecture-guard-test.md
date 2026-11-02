# [T-3] Architecture guard test

Milestone: M0 · Type: test · Layer: all · Size: small · Status: todo

## Goal

Fail the build when a crate breaks the dependency rule.

## Scope

- A test reads `cargo metadata` and checks normal (non-dev) dependencies:
  - `domain` depends on `thiserror` only.
  - `application` and every `infra-*` crate depend on no workspace crate except `domain`.
  - Only `infra-demucs` depends on `ort`.
  - No crate depends on `cli` or `web`.
- It runs in `cargo test --workspace`, so CI enforces it.

## Out of scope

- Checks on `use` paths or module layout.

## Acceptance criteria

- [ ] Passes on the T-1 skeleton.
- [ ] Each forbidden edge (e.g. `ort` in `domain`, `infra-storage` in `infra-audio`, `web` in `cli`) fails the test with a message naming both sides.

## Test plan

_Filled when the task moves to `planned`._

## Implementation plan

_Filled when the task moves to `planned`._

## Notes

None.
