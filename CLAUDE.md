# NoiseApp

Personal local web app + CLI (Windows + Linux; standalone executables or Docker)
that splits a song into stems
(vocals, drums, bass, other; optionally guitar, piano) and exports them
as WAV or FLAC. Later milestones: region export, noise reduction.

## Workflow rules (mandatory)

1. Order is always: plan -> tests -> code. No implementation before its
   plan is approved and its tests exist.
   Tasks that touch the web UI (`frontend/`) add a design step:
   plan -> design -> tests -> code. The design is a claude.ai Design
   artifact, approved by the user before tests are written; its link goes
   into the task file and the issue.
   Design inputs: `frontend/DESIGN.md` sets palette and type; the
   `design-taste-frontend` skill (taste-skill plugin, app-screen run) sets
   composition and house rules. On conflict, colours and fonts follow
   DESIGN.md, everything else follows the skill.
   Theme tokens: `frontend/src/theme/tokens.json` is the single source of
   truth for colour, type, spacing, radius, elevation and motion. It is
   derived from DESIGN.md and approved by the user in the first web UI
   task, before any screen is designed. `tokens.css` and `tokens.ts` are
   generated from it at build time and never edited by hand. Every design
   and every line of frontend code uses tokens only; a new value is added
   to tokens.json first (with user approval), never hard-coded. A lint
   check fails CI on raw colour, spacing, radius or font values outside
   the token files.
2. Tests are written from the plan, never derived from the code.
3. No task file or GitHub issue is created without the user approving its text.
4. docs/TASKS.md, docs/tasks/*.md and GitHub issues stay in sync.
   Every status change is applied to all of them.
5. Ask the user whenever a decision is not settled in docs/PLAN.md.
6. English for code comments, commit messages, docs, tasks and issues.
7. No commit trailers (no Co-Authored-By, no Signed-off-by).
8. Commit or push only when the user asks.
9. Commit messages follow Conventional Commits; scope is the crate name,
   e.g. `feat(domain): add AudioBuffer`.

## Architecture

Clean Architecture. Dependencies point inward only.

| Crate          | Role                                     | Depends on |
|----------------|------------------------------------------|------------|
| domain         | entities, value objects, ports (traits)  | none       |
| application    | use cases                                | domain     |
| infra-audio    | decode, encode, resample adapters        | domain     |
| infra-demucs   | Demucs ONNX separator adapter            | domain     |
| infra-storage  | .noiseproj repository                    | domain     |
| cli            | `noiseapp` binary: command line, composition root | all |
| web            | `noiseapp-web` binary: HTTP API (axum) + embedded frontend, composition root | all |

`domain` may depend on `thiserror` only. No audio, ML or filesystem crates.
Only `infra-demucs` depends on `ort`.

`frontend/` is a Svelte + TypeScript (Vite) app, built and embedded into `web`.
It talks to `web` over HTTP only.

## Tracking

- Task ID `T-N` (no zero padding, e.g. `T-7`), issue title `[T-N] <title>`, file `docs/tasks/T-N-<slug>.md`.
- Status flow: `todo -> planned -> tests-written -> in-progress -> done`
  (mirrored as `status:*` labels on the issue). UI tasks add `designed`
  between `planned` and `tests-written`.
- Each task plan states `Size: small | large`; the user confirms it.
- Milestones M0-M8 are GitHub Milestones.
- Labels: `type:feature|test|docs|chore`, `layer:domain|application|infra|cli|web|frontend|tools`.

## Branching

- One branch per milestone: `m<N>-<slug>` (e.g. `m0-foundation`).
- Small issues: commit directly to the milestone branch.
- Large issues: branch `t-<N>-<slug>` from the milestone branch, PR back into it.
- Milestone done: one PR from the milestone branch into `main`.

## Commands

To be filled in once the workspace exists.
