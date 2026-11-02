# [T-6] Domain audio ports

Milestone: M0 · Type: feature · Layer: domain · Size: small · Status: todo

## Goal

Ports for reading and writing audio, so use cases run against fakes.

## Scope

- `AudioDecoder`: path -> `AudioBuffer`.
- `AudioEncoder`: `AudioBuffer` + `ExportFormat` -> byte sink (file or zip entry).
- Errors: unsupported format, corrupt data, I/O.

## Out of scope

- Adapters (T-7 to T-9); other ports come with their milestones.

## Acceptance criteria

- [ ] An in-memory fake of each port can be written and used in a test.
- [ ] Callers can tell unsupported format, corrupt data and I/O errors apart.

## Test plan

_Filled when the task moves to `planned`._

## Implementation plan

_Filled when the task moves to `planned`._

## Notes

None.
