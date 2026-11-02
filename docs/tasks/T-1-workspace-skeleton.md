# [T-1] Workspace skeleton

Milestone: M0 · Type: chore · Layer: all · Size: small · Status: done

## Goal

Cargo workspace with all seven crates, so later tasks only add code.

## Scope

- Crates `domain`, `application`, `infra-audio`, `infra-demucs`, `infra-storage`, `cli`, `web` under `crates/`.
- `cli` builds the `noiseapp` binary, `web` builds `noiseapp-web`; the rest are libraries.
- Path dependencies exactly as in the CLAUDE.md table.
- `.gitignore`, `.gitattributes` (LF line endings, binary audio and ONNX files), `rustfmt.toml`, workspace clippy lints, `rust-toolchain.toml` pinning a stable release.

## Out of scope

- CI (T-2), dependency guard (T-3), domain code, `frontend/` (M5), `tools/export/` (M1).

## Acceptance criteria

- [x] `cargo build --workspace` passes on Windows and Linux and produces `noiseapp` and `noiseapp-web`.
- [x] `cargo fmt --check` and `cargo clippy --workspace --all-targets -- -D warnings` pass.
- [x] Crate dependencies match the CLAUDE.md table.

## Test plan

T-1 adds no test code; this checklist is its test. The dependency guard that keeps the graph correct later is T-3.

1. `cargo metadata --no-deps --format-version 1` lists exactly seven members: `domain`, `application`, `infra-audio`, `infra-demucs`, `infra-storage`, `cli`, `web`.
2. `cargo tree -e normal --depth 1 -p <crate>` shows exactly these workspace edges and no external dependencies:
   - `domain`: none
   - `application`, `infra-audio`, `infra-demucs`, `infra-storage`: `domain`
   - `cli`, `web`: `domain`, `application`, `infra-audio`, `infra-demucs`, `infra-storage`
3. `cargo build --workspace` exits 0 on Windows and on Linux (`rust:1.99.1` Docker image, separate target dir).
4. `noiseapp` and `noiseapp-web` (`.exe` on Windows) exist in `target/debug/` and exit with code 0.
5. `cargo fmt --all --check` exits 0.
6. `cargo clippy --workspace --all-targets -- -D warnings` exits 0.
7. `rustup show active-toolchain` inside the repo reports `1.99.1`.
8. `git ls-files --eol` shows `w/lf` for every text file.

## Implementation plan

1. Root `Cargo.toml`, virtual workspace:
   - `resolver = "3"`, explicit `members` list under `crates/`.
   - `[workspace.package]`: `version = "0.1.0"`, `edition = "2024"`, `rust-version = "1.99"`, `license = "MIT"`, `repository`, `publish = false`.
   - `[workspace.dependencies]`: one path entry per internal crate.
   - `[workspace.lints]`: `unsafe_code = "forbid"`; clippy `dbg_macro`, `todo`, `unimplemented` = `warn` (CI turns warnings into errors).
2. Every crate inherits package fields and lints. Libraries get `src/lib.rs` with a one-line crate doc. `cli` and `web` get `src/main.rs` with an empty `main`; bin names `noiseapp` and `noiseapp-web`.
3. `rust-toolchain.toml`: `channel = "1.99.1"`, components `rustfmt` and `clippy`, profile `minimal`.
4. `rustfmt.toml`: `newline_style = "Unix"`.
5. `.gitignore`: `/target`.
6. `.gitattributes`: `* text=auto eol=lf`; audio fixtures (`*.wav`, `*.flac`, `*.mp3`, `*.ogg`, `*.m4a`) and `*.onnx` marked `binary`.
7. `Cargo.lock` is committed, because the workspace ships binaries.
8. One commit on `m0-foundation`: `chore: add cargo workspace skeleton`.

## Notes

- 2026-11-03: all eight checks pass on Windows (MSVC) and Linux (`rust:1.99.1-slim` image).
