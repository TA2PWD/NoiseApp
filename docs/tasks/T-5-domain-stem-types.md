# [T-5] Domain stem types

Milestone: M0 · Type: feature · Layer: domain · Size: small · Status: todo

## Goal

Types for models, stems, regions and export formats.

## Scope

- `StemKind`: Vocals, Drums, Bass, Other, Guitar, Piano.
- `ModelSpec`: id, ordered stems, sample rate, segment length, overlap, member networks with per-stem weights.
- `StemSet`: one `AudioBuffer` per `StemKind`, same length and rate.
- `Region`: start/end in samples, `start < end <= length`.
- `ExportFormat`: `Wav(WavDepth)` (16, 24, 32f) or `Flac(FlacDepth)` (16, 24).

## Out of scope

- Model registry with URLs and hashes (M4), `Project` (M3).

## Acceptance criteria

- [ ] `ModelSpec` rejects an empty or duplicate stem list, overlap outside `[0, 1)`, a zero segment length, and member weights without one value per stem.
- [ ] `StemSet` rejects buffers with different lengths or sample rates.
- [ ] `Region` rejects `start >= end` and `end > length`.
- [ ] Unsupported depths cannot be expressed.

## Test plan

_Filled when the task moves to `planned`._

## Implementation plan

_Filled when the task moves to `planned`._

## Notes

None.
